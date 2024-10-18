### Vulnerability Report

**Product:** **com.ledvance.smartplus**

**Vendor:** **SYLVANIA Smart Home**

**Version:** **v3.0.3**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

1. **Project Description**  
   SYLVANIA Smart Home v3.0.3 is an application that allows users to make their homes smarter by pairing and controlling SYLVANIA SMART+ Bluetooth lights and accessories. It enables users to prepare and authenticate their fixtures and accessories, making them compatible with OpenAI Home, Amazon Alexa, and Apple HomeKit, thus illuminating their homes in a smarter way.

2. **Vulnerability Description**  
   The SYLVANIA Smart Home v3.0.3 app lacks access control restrictions or authentication mechanisms, enabling attackers to exploit leaked information from the APK file to download firmware. More specifically, due to inadequate access controls during the firmware update and download processes, potential attackers can analyze the code and data within the APK file to locate methods or classes related to firmware updates. By utilizing network request components such as retrofit2, they can successfully retrieve the complete firmware from the server without authorization. This creates serious security vulnerabilities, allowing attackers to analyze the firmware and extract potentially sensitive data like passwords, encryption keys, and configuration files. If leaked, this data can lead to system breaches, and attackers can also access the firmware's source code to modify it, thereby gaining full control over the devices.

3. **Vulnerability Reproduction**  
   Manual analysis of the APK file using tools like jadx enabled the identification of methods or classes in the app related to firmware updates. By analyzing the APK file's code and data, essential information was obtained regarding URLs through the network request APIs, primarily retrofit2, from classes such as `com.ledvance.smartplus.data.repository.Api2Service` and `com.ledvance.smartplus.utils.Env`. This led to a complete URL being constructed, as illustrated in Figures 1, 2, and 3.

   ![](https://s2.loli.net/2024/10/17/7ORpqN6BZKxgj4V.png)

   ![](https://s2.loli.net/2024/10/17/KPakHd7oeJhA261.png)

   ![](https://s2.loli.net/2024/10/17/9pR5TbvdmNXJLuG.png)

   By combining and accessing this information, potential attackers can obtain a txt file containing relevant URLs for the current app's firmware updates or downloads, as shown in Figure 4.

   ![](https://s2.loli.net/2024/10/17/L69uNYkTn3VMGZs.png)


   Based on the device number and URL from the firmware.txt file, attackers can successfully download firmware files without authorization, successfully acquiring a total of 115 firmware files, as depicted in Figure 5.

   ​                         ![](https://s2.loli.net/2024/10/17/mxeczkPZwnK5UHG.png)

4. **Vulnerability Impact**  
   The lack of necessary access control mechanisms on the server allows attackers to obtain complete firmware. By analyzing the firmware, they can extract potentially sensitive data like passwords, encryption keys, and configuration files. If this data is leaked, it can lead to system compromises. Furthermore, attackers can access the firmware's source code and modify the firmware, achieving full control over the devices.

5. **Remediation Suggestions**  

   - **Implement Access Control or Authentication Mechanisms:** Add permissions or authentication processes to relevant functions within the app.  
   - **Enforce Strict Permission Checks:** Ensure that all requests on the server pass strict authentication and authorization checks, permitting only authorized users to access firmware download links.  
   - **Validate Firmware Download Requests:** Implement verification of firmware download requests by checking the request source, device ID, user ID, etc., to ensure requests originate from legitimate devices and users.  
   - **Dynamic Firmware Download Links:** The server should dynamically generate firmware download links with a short expiration duration, rendering the links invalid after a specified time to prevent misuse.  

