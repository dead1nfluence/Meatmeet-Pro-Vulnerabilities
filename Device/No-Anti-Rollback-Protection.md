### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The ESP32 system on a chip (SoC) that powers the Meatmeet basestation was found to have anti-rollback protection disabled. Anti-rollback protection feature ensures that device only executes the application that meets the security version criteria as stored in its eFuse. So even though the application is trusted and signed by legitimate key, it may contain some revoked security feature or credential. Hence, device must reject any such application.

This, along with other vulnerabilities found, enables an attacker with physical access to the Meatmeet base station in the deployment of malicious code to the ESP32. As a result, the malicious code will be executed and the device will be taken over. 

### References 
- CWE-693: Protection Mechanism Failure

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART.
3. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
4. Using [espefuse](https://docs.espressif.com/projects/esptool/en/latest/esp32/espefuse/index.html) run: `espefuse dump`
5. Observe that Anti-Rollback Protections are disabled.
