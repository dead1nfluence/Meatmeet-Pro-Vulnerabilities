### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
As UART download mode is still enabled on the ESP32 chip on which the firmware runs, an adversary can dump the flash from the device and retrieve sensitive information such as details about the current and previous Wi-Fi network from the NVS partition. Additionally, this allows the adversary to reflash the device with their own firmware which may contain malicious modifications.

### References 
- CWE-693: Protection Mechanism Failure
- CWE-1191: On-Chip Debug and Test Interface With Improper Access Control

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the Meatmeet.
4. Connect to the device over UART with picocom, where * is the number associated with your USB-UART adapter: `sudo picocom /dev/ttyUSB* -b 115200`
5. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
6. The UART log will now display that the device is in download mode.

