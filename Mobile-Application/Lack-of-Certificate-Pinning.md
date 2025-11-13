### Affected Entity
- Meatmeet Pro Mobile Application v1.1.2.0 

### Impact
Due to a lack of certificate validation, all traffic from the mobile application can be intercepted. As a result, an adversary located "upstream" can decrypt the TLS traffic, inspect its contents, and modify the requests in transit. This may result in a total compromise of the user's account if the attacker intercepts a request with active authentication tokens or cracks the MD5 hash sent on login.

### References
-  CWE-295: Improper Certificate Validation 
