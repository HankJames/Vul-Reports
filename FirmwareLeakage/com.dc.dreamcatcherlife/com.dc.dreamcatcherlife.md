### Vulnerability Report

**Product:** **com.dc.dreamcatcherlife**

**Vendor:** **DreamCatcher Life**

**Version:** **v1.8.7**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

1. **Project Description**  
   DreamCatcher Life v1.8.7 is an application that allows remote control of various wireless smart devices. With quick and simple network connectivity, users can remotely manage and control a variety of smart devices in their homes. The app supports one-click control of multiple devices and can set multiple automated scenes, enabling users to enjoy a secure and worry-free smart home experience.

2. **Vulnerability Description**  
   Due to the lack of access control restrictions or authentication in the DreamCatcher Life v1.8.7 app, attackers can exploit leaked information from the APK file to download firmware. Specifically, the absence of access control during the firmware update and download process enables potential attackers to analyze the code and data within the APK file, identify methods or classes related to firmware updates, and utilize the network request component fuelKT to successfully obtain the complete firmware related to the app without authorization. This leads to serious security risks, as attackers can analyze the firmware to extract potentially sensitive data, such as passwords, encryption keys, and configuration files. If leaked, this information may lead to system compromise. Moreover, attackers can access the firmware source code, modify it, and thereby gain full control over the device.

3. **Vulnerability Reproduction**  
   Manual analysis of the APK file using tools like jadx reveals methods or classes related to firmware updates by examining the app's code and data. Utilizing the network request methods from fuelKT enables the extraction of relevant information. Constructing the parameters for network requests requires information such as token, as shown in Figure 2.
                    ![](https://s2.loli.net/2024/10/17/f4kjAxMdc8eaG5t.png)

   ![](https://s2.loli.net/2024/10/17/jNEu9eZnmCIib4M.png)

   Token and other information can be obtained by downloading the app on a mobile testing device, registering an account, and using Canary for packet capture, as illustrated in Figure 3.
                                                                       ![](https://s2.loli.net/2024/10/17/Vp2LmiGMg8vkUTF.png)

   Further extraction and analysis of this information allow potential attackers to obtain URLs related to the current app's firmware update or download. By performing a brute-force attack on the pid, attackers can successfully acquire valid pid values, as shown in Figure 4, and then proceed to download firmware for all devices, as demonstrated in Figure 5. This proves that the app's website lacks a user identity verification mechanism, facilitating firmware leakage and allowing potential attackers to successfully retrieve firmware without authorization, as depicted in Figure 6.
                                         ![](https://s2.loli.net/2024/10/17/ZLYtOHQl7E4hqyn.png)

   ![](https://s2.loli.net/2024/10/17/Ud7qmr89Mj43thG.png)

   ![](https://s2.loli.net/2024/10/17/Pwmv4AaI1YNdrl5.png)

4. **Vulnerability Impact**  
   Due to the server's lack of necessary access control mechanisms, attackers can obtain complete firmware from the server. By analyzing the firmware, they may extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this information could lead to system compromise, and attackers could also gain access to the firmware source code, modify it, and thereby achieve full control over the device.

5. **Remediation Suggestions**  
   Implement access control or authentication mechanisms for relevant functions.  
   Ensure that all requests undergo strict authentication and authorization checks on the server side. Only authorized users should be able to access firmware download links.  
   Validate firmware download requests by checking request sources, device IDs, user IDs, etc., to ensure that requests come from legitimate devices and users.
   The server can dynamically generate firmware download links with a short validity period, making the link expire after a certain time to prevent abuse.  
   Process parameters like Pid using secure encryption algorithms before storing and transmitting them.
