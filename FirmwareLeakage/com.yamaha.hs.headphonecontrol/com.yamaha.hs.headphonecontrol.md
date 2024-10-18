### **Vulnerability Report**

**Product:**  **com.yamaha.hs.headphonecontrol**

**Vendor:** **Yamaha**

**Version:** **v1.2.0**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

**1  ProjectInformation**

Yamaha HeadPhoneControl v1.2.0 allows users to customize functions and personalize settings forspecific models of Yamaha headphones and earphones. Supported models: YamahaTW-E5B, TW-ES5A, TW-E7B, TW-E3C, YH-E700B, etc.

**2  VulnerabilityDescription**

The Yamaha HeadPhone Control v1.2.0 APP(package name: com.yamaha.hs.headphonecontrol) lacks access controlrestrictions or authentication. Attackers can exploit information leaked in theAPK file to download the firmware. Specifically:

Due to the lack of access control forfirmware updates and downloads, potential attackers can analyze the APK file'scode and data to locate methods or classes related to firmware updates withinthe app. They can leverage functions related to network requests tosuccessfully obtain the complete firmware associated with the APP withoutauthorization or permission, leading to severe security risks. By analyzing thefirmware, attackers may obtain sensitive data such as passwords, encryptionkeys, and configuration files, which, if leaked, could compromise the system.They could even gain full control over the device by modifying the firmwarethrough access to its source code.

**3**  **VulnerabilityReproduction**

Through manualanalysis of the software APK file using tools like jadx, methods or classesrelated to firmware updates within the app were identified. By utilizingnetwork request-related methods, relevant information was obtained, as shown inFigures 1 and 2.

![](https://s2.loli.net/2024/10/17/hXUf6MkcJ2VDe8t.png)

![](https://s2.loli.net/2024/10/17/GqPAInXRxTehzQc.png)

![](https://s2.loli.net/2024/10/17/Nwo5QenfLTlhUk2.png)

Furtherextraction and analysis of this information enable potential attackers toconstruct and send network requests to obtain URLs related to current APPfirmware updates or downloads, thereby successfully retrieving the APP firmwarewithout authorization, as illustrated in Figure 3.

![](https://s2.loli.net/2024/10/17/BUf4xcXjeCDHtY7.png)

A total of 7firmware samples were successfully downloaded, as shown in Figure 4.

                                      ![](https://s2.loli.net/2024/10/17/LSwothGlKXQcVen.png)

Due to theserver's lack of necessary access control mechanisms, attackers can retrievethe complete firmware from the server. By analyzing the firmware, they canacquire sensitive data such as passwords, encryption keys, and configurationfiles, which could lead to system compromise. Additionally, by obtaining thefirmware source code, they could modify the firmware and gain full control overthe device.

**5**  **RemediationRecommendations**

Implementpermission control or authentication mechanisms for relevant functions.

Ensure that all requests undergo strictauthentication and authorization checks on the server side. Only authorizedusers should be able to access firmware download links.

Validatefirmware download requests, checking for request origin, device ID, user ID,etc., to ensure that requests come from legitimate devices and users.

The server candynamically generate firmware download links with a short expiration period;after expiration, the link becomes invalid to prevent misuse.

