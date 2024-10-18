### Vulnerability Report

**Product:** **com.ivyiot.IvySmart**

**Vendor:** **IVY Smart**

**Version:** **v4.5.0**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

1.**Project Description**  
IVY Smart v4.5.0 is an application designed to work with cameras for quick monitoring, historical playback, and wireless security monitoring. The software allows remote connection and configuration of device parameters and supports two-way voice intercom, video playback, and other functionalities.

2.**Vulnerability Description**  
Due to the lack of user identity verification mechanisms in the IVY Smart v4.5.0 app, attackers can exploit leaked information from the APK file along with the ID of a real physical device. By combining this with any user ID, they can successfully download the firmware. Specifically, the absence of authentication during the firmware update and download processes allows potential attackers to analyze the APK file's code and data. They can locate methods or classes in the app related to firmware updates and utilize related network request APIs to obtain complete firmware associated with the app without authorization. This poses serious security risks, as attackers can analyze the firmware to extract potentially sensitive data, such as passwords, encryption keys, and configuration files. If leaked, such data could lead to system compromise. Furthermore, attackers could gain access to the firmware source code, modify it, and thereby obtain full control over the device.

3.**Vulnerability Reproduction**  
Manual analysis of the APK file using tools like jadx helped identify methods or classes in the app related to firmware updates. By utilizing relevant APIs from files such as AuthIOTFirmwareUpdateEntity.java, information needed for constructing a firmware update network request was obtained, as shown in Figure 2.

![](https://s2.loli.net/2024/10/17/6uPW7h2DMEc3bgU.png)

![](https://s2.loli.net/2024/10/17/jas2gvuOpWtKDF3.png)

Further extraction and analysis of this information revealed that when requesting firmware updates, the app requires a physical device ID, openid, accessToken, and clientid. By combining this information with findings from the APK file analysis, URLs related to firmware updates or downloads for the current app can be obtained, allowing unauthorized access to the app's firmware.
The physical device ID was acquired by purchasing a real hardware device, specifically the "E3P Video Call Camera," model IPC-D11/WIFI. To ensure compatibility with the app, it was chosen based on its product parameters for "Aiwei IoT," as shown in Figure 3. This successfully retrieved the physical device ID: IC264C52UV2B52463ZZZCJB, illustrated in Figure 4.

![](https://s2.loli.net/2024/10/17/x4nfWgA3KjyBOUF.png)

![](https://s2.loli.net/2024/10/17/eBcqbx1ndFvHKwD.png)

Openid, accessToken, and clientid can be captured by downloading the app on a mobile testing device, registering an account, and using Canary for packet capture, as shown in Figure 5. Multiple accounts were registered, and packet captures were performed to obtain several valid pieces of information.

![](https://s2.loli.net/2024/10/17/NKvG1dLrWlax4Vg.png)

With the information obtained, a firmware update network request can be constructed using the known physical device ID and arbitrary user information, allowing firmware download from relevant websites, as shown in Figure 6. This demonstrates that the app's website lacks a user identity verification mechanism, leading to firmware leakage, allowing potential attackers to successfully obtain firmware without authorization.
                                 ![](https://s2.loli.net/2024/10/17/jGSIaFgCvZXiL2H.png)

Using the official URL, the firmware file 1767744531157409792.bin was successfully downloaded, as shown in Figure 7.

![](https://s2.loli.net/2024/10/17/gdCGvYiOLZu1oWy.png)

4.**Vulnerability Impact**  
Due to the server's lack of necessary identity verification mechanisms, attackers can obtain complete firmware from the server. By analyzing the firmware, they may extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this data could lead to system compromise, and attackers could gain access to the firmware source code, modify it, and thus achieve full control over the device.

5.**Remediation Suggestions**  
Implement access control or authentication mechanisms for relevant functions.  
Ensure that all requests undergo strict authentication and authorization checks on the server side. Only authorized users should be able to access firmware download links.  
Validate firmware download requests by checking request sources, device IDs, user IDs, etc., to ensure that requests come from legitimate devices and users.  
The server can dynamically generate firmware download links that have a short expiration time, rendering the links invalid after a set period to prevent abuse.  
