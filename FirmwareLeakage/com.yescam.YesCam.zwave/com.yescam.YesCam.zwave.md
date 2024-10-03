# Vulnerability Report - com.yescam.YesCam.zwave- Incorrect Access Control

**Product:** `com.yescam.YesCam.zwave` 

**Vendor:** [YESCAM](https://www.asmag.com.tw/)

**Version:** 1.0.2

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.yescam.YesCam.zwave` , developed by **YESCAM**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `Auto_upgrade_url` Field value in the `CameraList` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003201920434](https://s2.loli.net/2024/10/03/MUz7hGrkoDT3dwt.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://str.flirservices.com/versions.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: str.flirservices.com
   Content-Length: 0
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 868
   Content-Type: application/json
   Date: Thu, 03 Oct 2024 12:18:54 GMT
   Etag: "3655eca77c83591851e0abd87e92f2bb"
   Last-Modified: Tue, 06 Feb 2018 09:52:17 GMT
   Server: AmazonS3
   X-Amz-Id-2: IvqGCITnfB7EoLJ9mTR6U03aoOxVsrccbXu9aY60hoasmxmtXRt1Mk/fLHuy11CHBqNReG0pgfM=
   X-Amz-Request-Id: VS1MJ1WJQK7K6083
   
   {
     "vendor": "starvidia",
     "products": [
       {
         "model": "LNC104",
         "version": "030605",
         "url": "http://str.flirservices.com/LNC104/LNC104_20150624_030699.bin",
         "md5": "",
         "type": "regular",
         "source": "*"
       },
       {
         "model": "LNC204",
         "version": "030605",
         "url": "http://str.flirservices.com/LNC204/LNC204_20150624_030699.bin",
         "md5": "",
         "type": "regular",
         "source": "*"
       },
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003202136455](https://s2.loli.net/2024/10/03/Z7zf8Bb1wk4CTty.png)
   
   In total, we successfully downloaded **4 firmware** by exploiting the vulnerability.


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