# Vulnerability Report - com.theswitchbot.switchbot- Incorrect Access Control

**Product:** `com.theswitchbot.switchbot` 

**Vendor:** [SWITCHBOT INC](https://www.switch-bot.com/)

**Version:** 5.0.4

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.theswitchbot.switchbot` , developed by **SWITCHBOT INC**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `RELEASE_JSON_URL, DEBUG_JSON_URL` String value in the `MainContants` class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003191812214](https://s2.loli.net/2024/10/03/FAvBwh1R87YI624.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://notification.netvigator.com/version/wocaotech/release.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: notification.netvigator.com
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003192301203](https://s2.loli.net/2024/10/03/DehzPfMTGIEdxQb.png)
   
   In total, we successfully downloaded **7 firmware** by exploiting the vulnerability.


## Impact

An attacker can exploit this vulnerability to:

* Download the latest firmware before it is officially released.
* Analyze the firmware for vulnerabilities or proprietary information.
* Create modified firmware images with malicious code.
* Distribute compromised firmware updates to unsuspecting users.

## Remediation

* Implement proper access control on the firmware server, such as device ID and user ID verification.
* Enhance the app's firmware update mechanism to include authentication credentials in the requests.


## Disclaimer

This vulnerability report is provided for informational purposes only and should not be used for any malicious activities. The author is not responsible for any misuse of the information contained herein.