### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The ESP32 system on a chip (SoC) that powers the Meatmeet basestation was found to have anti-rollback protection disabled. The Secure Boot feature ensures that only authenticated software can execute on the device. The Secure Boot process forms a chain of trust by verifying all **mutable** software entities involved in the [Application Startup Flow](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/startup.html). 

This, along with other vulnerabilities found, enables an attacker with physical access to the Meatmeet base station in the deployment of malicious code to the ESP32. As a result, the malicious code will be executed and the device will be taken over. 

### References 
- CWE-693: Protection Mechanism Failure

### Replication Steps
1. Disassemble the Meatmeet Pro, exposing its internal circuit board.
2. Using probes and a USB-UART adapter, connect to the device over UART.
3. Put the device into [download mode by pulling IO9 low](https://docs.espressif.com/projects/esptool/en/latest/esp32/advanced-topics/boot-mode-selection.html).
4. Using [espefuse](https://docs.espressif.com/projects/esptool/en/latest/esp32/espefuse/index.html) run: `espefuse dump`
5. Observe that Anti-Rollback Protections are disabled.
