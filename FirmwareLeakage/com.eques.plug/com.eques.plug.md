# Vulnerability Report - com.eques.plug- Incorrect Access Control

**Product:** `com.eques.plug`

**Vendor:** [EQUES](http://www.eques.cn/?lang=en)

**Version:** 1.0.1

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.eques.plug` , developed by **EQUES**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `return` values in the `getBinURL`method provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003205648548](https://s2.loli.net/2024/10/03/TmpSEWJGuoiHbQA.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://www.ikonke.us/firmware/kit/kit.bin HTTP/1.0
   User-Agent: Fiddler
   Host: www.ikonke.us
   ```

   ```http
   GET http://www.kankunit.com/k2/kk-strip/user2.bin HTTP/1.0
   User-Agent: Fiddler
   Host: www.kankunit.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 3866628
   Accept-Ranges: bytes
   Content-Type: application/octet-stream
   Date: Thu, 03 Oct 2024 13:20:54 GMT
   Etag: "3b0004-53b5894227400"
   Last-Modified: Wed, 31 Aug 2016 06:55:12 GMT
   Server: Apache
   
      OpenWrt                 r45746                                       ,  j G iyni"                               |  
   
   *** FIDDLER: RawDisplay truncated at 128 characters. Right-click to disable truncation. ***
   ```

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 380952
   Accept-Ranges: bytes
   Content-Type: application/octet-stream
   Date: Thu, 03 Oct 2024 13:15:05 GMT
   Etag: "5fbf955c-5d018"
   Last-Modified: Thu, 26 Nov 2020 11:45:32 GMT
   Server: nginx
   
      @  @j    @!     R      > 2                                                                                  
   
   *** FIDDLER: RawDisplay truncated at 128 characters. Right-click to disable truncation. ***
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003212307687](https://s2.loli.net/2024/10/03/LfV9lS2ZYHsCQD8.png)

   In total, we successfully downloaded **6 firmware** by exploiting the vulnerability.

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