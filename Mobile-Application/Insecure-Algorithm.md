### Affected Entity
- Meatmeet Pro Mobile Application v1.1.2.0 

### Impact
The application uses an insecure hashing algorithm (MD5) to hash passwords. If an attacker obtained a copy of these hashes, either through exploiting cloud services, performing TLS downgrade attacks on the traffic from a mobile device, or through another means, they may be able to crack the hash in a reasonable amount of time and gain unauthorized access to the victim's account.

### References 
-  CWE-327: Use of a Broken or Risky Cryptographic Algorithm 
