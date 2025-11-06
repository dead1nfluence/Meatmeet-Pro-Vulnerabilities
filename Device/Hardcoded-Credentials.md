### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4

### Impact
The Meatmeet Pro was found to be shipped with hardcoded Wi-Fi credentials in the firmware, for the test network it was developed on. If an attacker retrieved this, and found the physical location of the Wi-Fi network, they could gain unauthorized access to the Wi-Fi network of the vendor. Additionally, if an attacker were located in close physical proximity to the device when it was first set up, they may be able to force the device to auto-connect to an attacker-controlled access point by setting the SSID and password to the same as which was found in the firmware file.

### References
- CWE-798: Use of Hard-coded Credentials

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART.
3. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
4. Dump the flash by running the following command in a terminal: `esptool -p /dev/ttyUSB0 -b 460800 read-flash 0 ALL flashed.bin`
6. Extract the NVS partition from the flash dump: `./esp32knife.py --chip esp32c3 load_from_file flash.bin`
7. On the extracted NVS partition run: `strings nvs_out.bin | grep "maxeye"`

**Results**

The Wi-Fi password for the test network used by the vendor are returned in cleartext.

