### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The ESP32 system on a chip (SoC) that powers the Meatmeet basestation device was found to lack Secure Boot. As a result, an attacker with physical access to the device can flash modified firmware to the device, resulting in the execution of malicious code upon startup. 

### References 
- CWE-693: Protection Mechanism Failure

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART.
3. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
4. Using [espefuse](https://docs.espressif.com/projects/esptool/en/latest/esp32/espefuse/index.html) run: `espefuse dump`
5. Observe that Secure Boot is set to enabled on the device.
