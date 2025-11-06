### Affected Entity
- Meatmeet Pro BBQ Thermometer v1.0.34.4


### Impact
The firmware on the ESP32 is not encrypted, when pulled from the device an adversary can retrieve the information contained within the NVS partition, including the credentials of the current and previous Wi-Fi networks. 
