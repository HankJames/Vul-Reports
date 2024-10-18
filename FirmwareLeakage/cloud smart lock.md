### Vulnerability Report

**Product:** **com_seamooncloud_cloudsmartlock_v2.0.1**

**Vender:** **Cloud smart lock**

**Version:** **v2.0.1**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

**Affiliation:** **System Security Laboratory, College of Cyberspace Security, Shandong University**

**Contact:** wzl2022@mail.sdu.edu.cn  

---

1. **Project Description**  
   Cloud smart lock v2.0.1 is a smart door lock application designed to manage and operate cloud smart locks. Its main features include adding and managing locks, one-click unlocking, QR code shared unlocking, automatic unlocking, as well as viewing unlocking status and unlocking history.

2. **Vulnerability Description**  
   Due to the lack of access control restrictions or authentication in the app, attackers can exploit information leaked from the APK file in the modules com.seamooncloud.cloudsmartlock.models.OperatorApi and com.seamooncloud.cloudsmartlock.models.http.NetworkConstant to bind physical devices. Specifically, the absence of access control allows potential attackers to analyze the code and data in the APK file, obtaining a complete URL that is leaked within the APK, which can invoke an API for binding physical devices. Attackers can find a valid sn value through brute force attacks and construct requests to use the app for binding to unknown devices, leading to significant harm.

3. **Vulnerability Reproduction**  
   Using the jadx tool to analyze the APK file, the baseUrl information for the app's network connection can be found in the module com.seamooncloud.cloudsmartlock.models.http.NetworkConstant, while subUrl information related to physical device operations can be found in the module com.seamooncloud.cloudsmartlock.models.OperatorApi. Combining both pieces of information can yield a valid URL for access. 
                                          ![1](https://s2.loli.net/2024/10/17/84YZnOtcx13VIv6.png)

   ![](https://s2.loli.net/2024/10/17/1wDX4rF7OMIW2hz.png)

   When accessing this URL, using Fiddler Everywhere for packet capture allows the discovery of the parameter information required for request construction, such as token, as shown in Figure 3. By registering on the Cloud smart lock v2.0.1 app on a mobile device and using Canary to capture packets, the corresponding token value for the registered user and other parameter values can be obtained, as shown in Figure 4. 

   ​                                                                            ![](https://s2.loli.net/2024/10/17/hsQBzCDItEjRPyn.png)

   Tests show that the sn value can be obtained through brute force attack by continuous attempts, revealing suitable sn values. When valid additional parameter values such as version and mark are provided, the sent request can be utilized for the current account to bind to unknown devices, as illustrated in Figure 5. After binding, attackers can use the app to collect information and control operations regarding the physical device, as shown in Figure 6.

   ![](https://s2.loli.net/2024/10/17/VtEqgioGYUAMcP6.png)

   ![](https://s2.loli.net/2024/10/17/6axuzhPQJ7O4WTC.png)

   ![](https://s2.loli.net/2024/10/17/sCetQ2KPV8YiF5x.png)

4. **Vulnerability Impact**  
   Attackers can arbitrarily bind physical devices and use the app to control these devices and collect information, resulting in significant harm.

5. **Remediation Suggestions**  
   Implement access control or authentication mechanisms for relevant functions.  
   Ensure that all requests undergo strict authentication and authorization checks on the server side. Only authorized users should be able to perform device binding.  
   Avoid storing and transmitting the sn value in plaintext; instead, utilize secure encryption algorithms to prevent brute force attacks effectively.
