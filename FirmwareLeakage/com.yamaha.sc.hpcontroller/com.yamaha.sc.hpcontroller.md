### **Vulnerability Report**

**Product:** **com.yamaha.sc.hpcontroller**

**Vendor:** **Yamaha**

**Version:** **v1.6.7**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

**1. Project Description**  
Yamaha HeadPhones Controller v1.6.7 allows users to easily manage their Yamaha headphones and device connections. Supported models include: TW-E3A, TW-E3B, EP-E30A, EP-E50A, EP-E70A, YH-E500A, YH-E700A, YH-L700A.

**2. Vulnerability Description**  
The Yamaha HeadPhones Controller v1.6.7 app (package name: com.yamaha.sc.hpcontroller) lacks access control restrictions and authentication mechanisms. This allows attackers to exploit leaked information from the APK file to download firmware. Specifically, the lack of access control over the firmware update and download processes enables potential attackers to analyze the code and data within the APK file. By locating methods or classes related to firmware updates within the app and utilizing associated network request functions, they can successfully retrieve the complete firmware related to the app without authorization, leading to serious security risks. Attackers can analyze the firmware to extract potentially sensitive data, such as passwords, encryption keys, and configuration files. If leaked, this data could lead to system breaches, and attackers could access the firmware source code, modify it, and gain full control over the devices.

**3. Vulnerability Reproduction**  
Manual analysis of the APK file using tools like JADX involved examining the code and data to identify methods or classes related to firmware updates, as well as utilizing associated network request functions to obtain relevant information, as shown in Figures 1 and 2.

![](https://s2.loli.net/2024/10/17/hXUf6MkcJ2VDe8t.png)

![](https://s2.loli.net/2024/10/17/GqPAInXRxTehzQc.png)

![](https://s2.loli.net/2024/10/17/Nwo5QenfLTlhUk2.png)

By further extracting and analyzing this information, potential attackers can construct and send network requests to obtain URLs related to the current app's firmware updates or downloads, as shown in Figure 3.

![](https://s2.loli.net/2024/10/17/BUf4xcXjeCDHtY7.png)

A total of 17 firmware samples were successfully downloaded, as illustrated in Figure 4.

![](https://s2.loli.net/2024/10/17/LSwothGlKXQcVen.png)

**4. Vulnerability Impact**  
The absence of necessary access control mechanisms on the server allows attackers to obtain complete firmware. By analyzing the firmware, they can extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this data may lead to system compromises, and attackers could access the firmware source code, modify it, and gain full control over the devices.

**5. Remediation Suggestions**  

- **Add Access Control or Authentication Mechanisms:** Implement permission controls or authentication systems for relevant functions.  
- **Ensure Strict Authentication Checks:** Make sure that all requests on the server undergo strict authentication and authorization checks. Only authorized users should be permitted to access firmware download links.  
- **Validate Firmware Download Requests:** Implement verification for firmware download requests, such as checking the request source, device ID, user ID, etc., to ensure that requests come from legitimate devices and users.  
- **Dynamic Firmware Download Links:** Servers should dynamically generate firmware download links and set a short expiration period so that links become invalid after a specified time to prevent misuse.
