### Vulnerability Report

**Product:** **com.gooclient.anycam.neye3ctwo**

**Vendor:** **Huachuang Tech.**

**Version:** **v4.5.2.0**

**Vulnerability Type: Key Leakage**

**Severity:** **High**

---

1. **Project Description**  
   neye3c v4.5.2.0 is an application designed to work with cameras, enabling quick monitoring, historical playback, and wireless security monitoring. The software allows remote connection and configuration of device parameters, as well as two-way intercom, video playback, and other functionalities.

2. **Vulnerability Description**  
   In the neye3c v4.5.2.0 app, specifically in the `com.gooclient.anycam.activity.settings.update.UpdateFirmwareActivity2`, the function responsible for obtaining new versions uses a hard-coded encryption key when calling Myhttp. This implementation exposes real values for the firmware URL and related parameters in the constructed HTTP requests. In other words, attackers can leverage this known information to craft legitimate yet unauthorized network requests to obtain critical files or data, such as firmware. Furthermore, knowing the key allows attackers to create and encrypt their own parameters, leading to the construction of network requests tailored to their needs, which is extremely dangerous.

3. **Vulnerability Reproduction**  
   Manual analysis of the APK file using tools like JADX revealed that in the `com.gooclient.anycam.activity.settings.update.UpdateFirmwareActivity2` implementation, the encryption operation for obtaining new versions uses a hard-coded key. This allows for direct retrieval of the complete key information.
                                        ![](https://s2.loli.net/2024/10/17/nDA9vYU25SJgapx.png)

   Based on this key, attackers can decrypt plaintext information at will or use the hard-coded key for encryption to forge parameters, such as downloading firmware without proper authentication. When a user binds the app to the device, parameters are captured, revealing information like {"AppCom":"Ong4EQ==","SolCom":"PW0i","ReleaseTime":"Rg1TZ0fnxVI=","HDVersion":"OG02EliGo1ZqhbSq"}, as shown in Figure 2.

   ![](https://s2.loli.net/2024/10/17/UcIo4ANeWV3FGps.png)

   Using a Python RC4 decryption script with the hard-coded key allows decryption of the ciphertext, resulting in the original URL as shown in Figure 3 and the original parameter information illustrated in Figure 4.

   ![](https://s2.loli.net/2024/10/17/b47xGcjFEOh1keH.png)

   ![](https://s2.loli.net/2024/10/17/uStGxb7VTcKP6md.png)

   With this information, potential attackers can construct network requests to update firmware, thus obtaining URLs associated with the current app’s firmware update or download, allowing unauthorized access to the app's firmware, as seen in Figure 5.
                                   ![](https://s2.loli.net/2024/10/17/kp74PmlSFIYvJW6.png)

4. **Vulnerability Impact**  
   The use of the same encryption key "Goolink2014" for multiple data points weakens security. Attackers can analyze the code to directly retrieve this key, allowing them to decrypt sensitive data, leading to information leakage. They can use the knowledge of this key to craft legitimate yet unauthorized network requests for critical files, including firmware. Moreover, with the key in hand, attackers could create their own parameters and encrypt them, facilitating the formation of customized network requests, which is highly perilous.

5. **Remediation Suggestions**  

   - **Use Configuration Files or Environment Variables for Key Storage:** Instead of hard-coding the key directly in the code, store it in configuration files or environment variables. This practice allows for easier key changes and reduces security risks.
   - **Implement a Key Management System:** Utilize a professional key management system to manage the encryption keys, which can better control access permissions and the scope of key usage.
