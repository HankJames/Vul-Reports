# Vulnerability Report - com.fermax.vida- Incorrect Access Control

**Product:** `com.fermax.vida` 

**Vendor:** [Fermax Asia Pacific Pte Ltd](https://www.fermax.com/)

**Version:** 2.4.6

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.fermax.vida` , developed by Fermax, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The  `firmware_link` String value in class `BuildConfig` provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003005320613](https://s2.loli.net/2024/10/03/tIasinPFOu6rXT8.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://www.starvedia.com/AutoUpgrade/HttpversionsFermax.json HTTP/1.1
   Host: www.starvedia.com
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 1918
   Accept-Ranges: bytes
   Content-Type: application/json
   Date: Wed, 02 Oct 2024 16:52:14 GMT
   Etag: "77e-60f06e786b1a5"
   Last-Modified: Tue, 16 Jan 2024 02:29:48 GMT
   Server: Apache/2.4.6 (CentOS) PHP/5.4.16
   
   {"vendor":"Fermax",
   "products":[
   {"model":"IC731z",
   "version":"030724",
   "url":"http://www.starvedia.com/AutoUpgrade/Firmware/IC731z-cloud-030724-n.f",
   "md5":"",
   "type":"regular",
   "source":"*",
   "devices":"*",
   "cfgVersion":"0",
   "cfgUrl":"*"
   },
   {"model":"IC731zp",
   "version":"040120",
   "url":"http://www.starvedia.com/AutoUpgrade/Firmware/IC731zp-cloud-040120-n.f",
   "md5":"",
   "type":"regular",
   "source":"*",
   "devices":"*",
   "cfgVersion":"0",
   "cfgUrl":"*"
   },
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003005337691](https://s2.loli.net/2024/10/03/dIawgWJ2LmYrPxS.png)

   


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