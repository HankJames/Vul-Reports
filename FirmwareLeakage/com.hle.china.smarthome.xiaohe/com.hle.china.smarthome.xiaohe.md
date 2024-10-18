#### **Vulnerability Report**

**Product:** **com.hle.china.smarthome.xiaohe**

**Vendor:** **XIAO HE Smart**

**Version:** **v4.3.1**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

##### 1.Problem Statement  

XIAO HE Smart v4.3.1 is a secure and reliable professional smart home application that supports remote management and control of various smart devices. It integrates with mainstream smart speakers both domestically and internationally, catering to various living scenarios to create a safe and worry-free smart home experience for users. The supported products include various lighting, appliances, and security devices.

##### 2.Vulnerability Description  

Due to the lack of access control restrictions in the XIAO HE Smart v4.3.1 APP, attackers can exploit leaked information in the APK file to download the official flashing tool. This tool can be used to set up and modify the physical devices associated with the APP. Specifically, the absence of access control for firmware updates and the download process allows potential attackers to analyze the code and data in the APK file, locate methods or classes in the APP related to firmware updates, and utilize network request methods to obtain a complete set of flashing tools and tutorials related to the hardware. These materials are theoretically accessible only to developers or authorized personnel, indicating that the APP lacks necessary access control mechanisms.

##### 3.Vulnerability Reproduction  

Manual analysis of the software's APK file was performed using tools such as jadx. By analyzing the code and data within the APK file, methods or classes related to firmware updates were identified, and relevant APIs for network requests were utilized to gather pertinent information.  

![](https://s2.loli.net/2024/10/17/xJtudLDQUK54zGE.png)

![](https://s2.loli.net/2024/10/17/gdTw9AHaQ74jPpJ.png)

![](https://s2.loli.net/2024/10/17/RTlZMCbH2xwDJWY.png)

By further extracting and analyzing this information, parameter values for deviceCode, version, and usercode were captured.  
When binding the APP with the lighting device "Xiao He Le Jia RGB Light Strip 21W," it was found that it does not support firmware updates. However, when a network request was constructed with the devicecode and version information filled in, the developer's flashing tools and a complete tutorial could be successfully obtained.  
Due to the lack of necessary access control mechanisms in the APP, attackers can acquire the full set of flashing tools and tutorials without any restrictions. This access is theoretically intended only for developers or authorized personnel, enabling the potential for harmful operations on the corresponding hardware, such as re-flashing.  

![](https://s2.loli.net/2024/10/17/4TJoYvkqROuS85y.png)

![](https://s2.loli.net/2024/10/17/rPmSXlNx48GiITa.png)

![](https://s2.loli.net/2024/10/17/ArHcjZCIpLqfyJx.png)

##### 4.Vulnerability Impact  

Due to the APP's lack of necessary access control mechanisms, attackers can obtain a complete set of flashing tools and tutorials without any restrictions. These materials are theoretically available only to developers or authorized personnel, allowing them the potential to perform unapproved re-flashing and other operations on the hardware.

##### 5.Remediation Plan  

The official flashing tools should not be accessible to users. It is recommended to store links to similar development files or tools in an encrypted format within the APK file and to implement user authentication mechanisms for any such download requests, ensuring that only authorized users can access relevant features.