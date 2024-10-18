### Vulnerability Report

**Product:** **com.gooclient.anycam.neye3ctwo**

**Vendor:** **Huachuang Tech**

**Version:** **v4.5.2.0**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

1. **Project Description**  
   neye3c v4.5.2.0 is an application designed to work with cameras for quick monitoring, historical playback, and wireless security monitoring. The software allows remote connection and configuration of device parameters, as well as two-way intercom, video playback, and other functionalities.

2. **Vulnerability Description**  
   Due to the lack of access control restrictions or authentication in the neye3c v4.5.2.0 app, attackers can exploit leaked information from the APK file to download firmware. Specifically, the absence of access control during the firmware update and download processes allows potential attackers to analyze the APK file's code and data. By locating methods or classes related to firmware updates in the app and utilizing network request methods like okHttpPost, they can successfully obtain complete firmware related to the app without authorization. This poses serious security risks, as attackers can analyze the firmware to extract potentially sensitive data, such as passwords, encryption keys, and configuration files. If leaked, this data could lead to system compromise, and attackers could also gain access to the firmware source code, modify it, and thereby obtain full control over the device.

3. **Vulnerability Reproduction**  
   Manual analysis of the APK file using tools like Jadx helped identify methods or classes in the app related to firmware updates. Direct URL information was obtained, as shown in Figure 1. It was noted that the POST request utilizes okHttp and requires parameters such as "AppCom," "SolCom," "ReleaseTime," and "HDVersion." These parameters are encrypted using the RC4 method; however, the key is stored in plaintext as "Goolink2014."

   ![](https://s2.loli.net/2024/10/17/nDA9vYU25SJgapx.png)

   To obtain the parameters required for constructing the firmware update network request, a physical device associated with the app, the "Network Eye LCD Screen Pan-Tilt Camera," model "S6-LCD," was purchased, as shown in Figure 2.

   ![](https://s2.loli.net/2024/10/17/ScmL7GnA9XgK8EQ.png)

   During the binding process between the app and the device, parameters were captured, obtaining information such as {"AppCom":"Ong4EQ==","SolCom":"PW0i","ReleaseTime":"Rg1TZ0fnxVI=","HDVersion":"OG02EliGo1ZqhbSq"}, as illustrated in Figure 3.

   ![](https://s2.loli.net/2024/10/17/UcIo4ANeWV3FGps.png)

   Using a Python RC4 decryption script with the hard-coded key allowed for the decryption of the ciphertext, resulting in the original URL shown in Figure 4, along with the original parameter information represented in Figure 5.

   ![](https://s2.loli.net/2024/10/17/b47xGcjFEOh1keH.png)

   ![](https://s2.loli.net/2024/10/17/uStGxb7VTcKP6md.png)

   With this information, potential attackers can construct network requests to update firmware, obtaining URLs related to the current app's firmware update or download, allowing unauthorized access to the app's firmware, as shown in Figure 6.

   ![image-20241017200327538](./../../../../AppData/Roaming/Typora/typora-user-images/image-20241017200327538.png)

4. **Vulnerability Impact**  
   Due to the server's lack of necessary access control mechanisms, attackers can obtain complete firmware from the server. By analyzing the firmware, they may extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this data could lead to system compromise, and attackers could gain access to the firmware source code, modify it, and thus achieve full control over the device.

5. **Remediation Suggestions**  
   Implement access control or authentication mechanisms for relevant functions.  
   Ensure that all requests undergo strict authentication and authorization checks on the server side. Only authorized users should be able to access firmware download links.  
   Validate firmware download requests by checking request sources, device IDs, user IDs, etc., to ensure that requests come from legitimate devices and users.  
   The server can dynamically generate firmware download links that have a short expiration time, rendering the links invalid after a set period to prevent abuse.  
