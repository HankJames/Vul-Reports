### Vulnerability Report

**Product:** **com.yingsheng.nadai**

**Vendor:** **Wear Sync**

**Version:** **v1.2.0**

**Vulnerability Type: Incorrect Access Control**

**Severity:** **High**

---

**1. Project Description**  
Wear Sync v1.2.0 is a free lifestyle application designed to help users track and achieve their health and fitness goals. It simplifies health management processes by collecting key information about users' workouts, sleep quality, and stress levels. The app can synchronize this information to users’ wearable devices and smart devices, providing a comprehensive view of their health.

**2. Vulnerability Description**  
Due to the lack of access control restrictions or authentication in the app, attackers can exploit leaked information from the APK file to download firmware. Specifically, the absence of authentication mechanisms during the firmware update and download processes allows potential attackers to analyze the code and data within the APK file. They can locate methods or classes related to firmware updates and utilize network request-related APIs. By leveraging compatible physical devices, they can successfully obtain the complete firmware related to the app without authorization, leading to significant security risks. Attackers can analyze the firmware to extract potentially sensitive data, such as passwords, encryption keys, and configuration files. If leaked, this data may lead to system compromise, and attackers could access the firmware source code, modify the firmware, and gain complete control over the devices.

**3. Vulnerability Reproduction**  
Manual analysis of the APK file using tools like JADX revealed that it uses the HAYLOU brand SDK, suggesting that it can interact with or perform functions related to HAYLOU devices.  

Further analysis of the code implementation showed that URL information related to firmware downloads or updates for the app can be obtained from files such as `com.liesheng.module_device.setting.viewmodel.OtaCheckViewModel$checkDeviceFwUpdate$1` and `com.liesheng.module_device.repository.DeviceService`, as illustrated in Figures 1, 2, and 3.

![](https://s2.loli.net/2024/10/17/gx6mGUiOd5YCLWV.png)

![](https://s2.loli.net/2024/10/17/rfxWVMO8EbF4BJq.png)

![](https://s2.loli.net/2024/10/17/aERDv5BApHZnQOd.png)

![](https://s2.loli.net/2024/10/17/aQNu37kJRiFsCYV.png)

Using two HAYLOU physical devices, HAYLOU Solar Plus and Watch 2Pro, the network packets exchanged between the app and the server were captured with Fiddler Everywhere during the process of binding the two devices. This allowed for the retrieval of parameters involved in the firmware update network request. Utilizing this information, potential attackers could actively construct and send network requests to obtain URLs related to the current app's firmware updates or downloads, thus successfully acquiring the app's firmware without authorization.

For Watch 2Pro, updates are primarily determined based on parts of deviceId, projectNo, and versionNo. After constructing the network request, the firmware download URL was successfully obtained, as shown in Figure 5.

![](https://s2.loli.net/2024/10/17/csJvapXCi63Ve8U.png)

![](https://s2.loli.net/2024/10/17/3SWVPf9B4kQdKTG.png)

For HAYLOU Solar Plus, after constructing the network request, the firmware download URL was successfully obtained, as shown in Figure 7.

![](https://s2.loli.net/2024/10/17/qobxhmikRrdM9tE.png)

![](https://s2.loli.net/2024/10/17/LjXNViRIA9J83qd.png)

**4. Vulnerability Impact**  
The absence of necessary access control mechanisms on the server allows attackers to retrieve complete firmware. By analyzing the firmware, they can extract potentially sensitive data such as passwords, encryption keys, and configuration files. If leaked, this data can lead to system compromise, and attackers could access the firmware source code, modify the firmware, and gain complete control over the devices.

**5. Remediation Suggestions**  

- **Add Access Control or Authentication Mechanisms:** Implement permission controls or authentication processes for relevant functions.  
- **Ensure Strict Authentication Checks:** Make sure that all requests on the server undergo strict authentication and authorization checks. Only authorized users should be able to access firmware download links.  
- **Validate Firmware Download Requests:** Verify firmware download requests by checking the request source, device ID, user ID, etc., to ensure that requests come from legitimate devices and users.  
- **Generate Dynamic Firmware Download Links:** The server should dynamically generate firmware download links with a short expiration time, causing the links to become invalid after a specified duration to prevent misuse.  
