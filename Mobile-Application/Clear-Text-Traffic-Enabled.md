### Affected Entity
- Meatmeet Pro Mobile Application v1.1.2.0 

### Impact
The mobile application is configured to allow clear text traffic to all domains and communicates with an API server over HTTP. As a result, an adversary located "upstream" can intercept the traffic, inspect its contents, and modify the requests in transit. This may result in a total compromise of the user's account if the attacker intercepts a request with active authentication tokens or cracks the MD5 hash sent on login.

### References
-  CWE-319: Cleartext Transmission of Sensitive Information 
