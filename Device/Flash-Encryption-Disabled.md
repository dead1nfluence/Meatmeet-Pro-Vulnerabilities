### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The firmware on the basestation of the Meatmeet is not encrypted. An adversary with physical access to the Meatmeet device can disassemble the device, connect over UART, and retrieve the firmware dump for analysis. Within the NVS partition they may discover the credentials of the current and previous Wi-Fi networks. This information could be used to gain unauthorized access to the victim's Wi-Fi network. 

### References 
-  CWE-311: Missing Encryption of Sensitive Data

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART.
3. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
4. In a terminal run the following command: `esptool -p /dev/ttyUSB0 -b 460800 read-flash 0 ALL reflashed.bin`
5. To retrieve the Wi-Fi credentials stored on the device run: `strings | grep <YOUR WIFI PASSWORD>`
