### Vulnerability Report

**Product:** **com.ruochan.dabai**

**Vendor:** **RuoChan Smart**

**Version:** **v4.4.7**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

**1. Project Description**  
RuoChan Smart v4.4.7 is a smart lock management platform that allows users to conveniently and easily connect with smart locks via their mobile phones.

**2. Vulnerability Description**  
The Ruochan Smart v4.4.7 app lacks necessary access control restrictions and authentication mechanisms, enabling attackers to exploit leaked information from the APK file to download firmware. Specifically, due to the absence of access control over the firmware update and download processes, potential attackers can analyze the code and data within the APK file, locate methods or classes related to firmware updates, and utilize the Retrofit structure of network requests. This allows them to successfully obtain the complete firmware related to the app without authorization or permission, leading to severe security risks. Analyzing the firmware could expose sensitive data, such as passwords, encryption keys, and configuration files. If leaked, this data may result in system breaches, and attackers could access the firmware source code, modify it, and subsequently gain full control over the devices.

**3. Vulnerability Reproduction**  
Manual analysis of the APK file using tools like JADX involved examining the code and data to identify methods or classes associated with firmware updates. By utilizing the relevant Retrofit network request methods within `com.ruochan.dabai.netcore.NetworkExtraService` and `com.ruochan.dabai.Constant`, specific information about URLs can be obtained. By using the values from BASE_URL_T6 and VersionInfo, a complete URL can be derived, as depicted in Figures 1, 2, and 3.

![](https://s2.loli.net/2024/10/17/Z3vmzkCY9AIHnLT.png)

![](https://s2.loli.net/2024/10/17/I7lQurPzFmqiYes.png)

![](https://s2.loli.net/2024/10/17/KYCnNMawy1pdSc2.png)

Through configuration files within the APK, the model name of the device related to the app can be retrieved, highlighted in blue in Figure 4. This information can be filled into the constructed network request for checking firmware updates to find the corresponding device firmware.

![](https://s2.loli.net/2024/10/17/wBOPCE9NYVkUub2.png)

Utilizing the obtained URL, potential attackers can acquire URLs related to the current app’s firmware updates or downloads, thereby obtaining the app’s firmware without authorization, illustrated in Figure 5.

![](https://s2.loli.net/2024/10/17/9evGJq8EUQIukYf.png)

Ultimately, a total of 62 firmware samples were successfully downloaded, as shown in Figure 6. This evidences that the app's website lacks a user identity verification mechanism, leading to firmware leakage. Attackers could successfully obtain firmware without permission.

![](https://s2.loli.net/2024/10/17/DTns7BzMmPYlAf8.png)

**4. Vulnerability Impact**  
The server's absence of necessary access control mechanisms allows attackers to retrieve complete firmware. By analyzing the firmware, they can extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this data may result in system breaches, and attackers could access the firmware source code, modify it, and gain full control over the devices.

**5. Remediation Suggestions**  

- **Add Access Control or Authentication Mechanisms:** Implement permission controls or authentication for relevant functions.  
- **Ensure Strict Authentication Checks:** Make sure that all requests on the server undergo strict authentication and authorization checks. Only authorized users should access firmware download links.  
- **Validate Firmware Download Requests:** Implement validation for firmware download requests by checking the request source, device ID, user ID, etc., to ensure that requests come from legitimate devices and users.  
- **Dynamic Firmware Download Links:** The server should dynamically generate firmware download links and set short expiration periods so links become invalid after a specified timeframe to prevent misuse.  
