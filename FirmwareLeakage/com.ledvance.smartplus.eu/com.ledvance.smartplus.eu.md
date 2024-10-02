# Vulnerability Report - com.ledvance.smartplus.eu- Incorrect Access Control

**Product:** `com.ledvance.smartplus.eu` 

**Vendor:** [LEDVANCE](https://www.ledvance.com/)

**Version:** 2.1.10

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.ledvance.smartplus.eu` , developed by LEDVANCE, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `checkFirmwareUpdate, checkFirmwareUpdateForAmazon, checkFirmwareUpdateForAppleHomeKit, checkFirmwareUpdateForGoogle` methods in the `Api2Service` class and the `FIRMWARE_URL` String value in class `Env.Companion` provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003003328039](https://s2.loli.net/2024/10/03/u6nBlHpi7yX2ODk.png)

   ![image-20241003003308862](https://s2.loli.net/2024/10/03/TtILim8JKBk9rDN.png)

   ![image-20241003003141781](https://s2.loli.net/2024/10/03/nd3iCeJXgrcpquW.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh.txt HTTP/1.1
   Host: apps.ledvanceus.com
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/plain
   Last-Modified: Mon, 08 Jan 2024 01:26:08 GMT
   Accept-Ranges: bytes
   ETag: "94c9e5a7d141da1:0"
   Server: Microsoft-IIS/7.5
   X-Powered-By: ASP.NET
   Date: Wed, 02 Oct 2024 16:35:17 GMT
   Content-Length: 14386
   
   {
   "SYLVANIA A19 C": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19C-M_2.0.05.bin","version": "2.0.5","deviceTypeId": "sylvania-a19-c"},
   "A19 C": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19C-M_2.0.05.bin","version": "2.0.5","deviceTypeId": "sylvania-a19-c"},
   
   "SYLVANIA A19 C1": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19C1-M_2.0.06.bin","version": "2.0.6","deviceTypeId": "sylvania-a19-c1"},
   "A19 C1": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19C1-M_2.0.06.bin","version": "2.0.6","deviceTypeId": "sylvania-a19-c1"},
   
   "SYLVANIA A19 F": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19F-M_2.0.05.bin","version": "2.0.5","deviceTypeId": "sylvania-a19-f"},
   "A19 F": {"fwFile": "https://apps.ledvanceus.com/firmware/ble/pd/eu/mesh/A19F-M_2.0.05.bin","version": "2.0.5","deviceTypeId": "sylvania-a19-f"},
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003003721696](https://s2.loli.net/2024/10/03/u7cS6Lhgj1UriI9.png)

   Downloaded total 182 firmware.


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