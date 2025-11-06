### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The ESP32 system on a chip (SoC) that powers the device was found to have JTAG enabled. By leaving JTAG enabled on an ESP32 in a commercial product an attacker with physical access to the device can connect over this port and reflash the device's firmware with malicious code which will be executed upon running. As a result, the victim will lose access to the functionality of their device and the attack may gain unauthorized access to the victim's Wi-Fi network by re-connecting to the SSID defined in the NVS partition of the device.

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART and force the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
3. Using [espefuse](https://docs.espressif.com/projects/esptool/en/latest/esp32/espefuse/index.html) run: `espefuse dump`
4. Observe that JTAG is still enabled on the device.
