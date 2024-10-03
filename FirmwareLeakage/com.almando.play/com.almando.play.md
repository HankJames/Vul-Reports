# Vulnerability Report - com.almando.play- Incorrect Access Control

**Product:** `com.almando.play` 

**Vendor:** [almando GmbH](http://www.almando.com/)

**Version:** 1.8.2

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.almando.play` , developed by **almando GmbH**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `ONLINE_JSON_URL,ONLINE_JSON_URL_INTEGRATION, ONLINE_JSON_EXPERT_URL` Field values in the `DownloadManager` class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003154311838](https://s2.loli.net/2024/10/03/Nces6MvI2SAQKGC.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://almando.com/media/firmware/almando-firmware-expert.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: almando.com
   
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 07:44:14 GMT
   Server: Apache/2.4.61 (Debian)
   Last-Modified: Wed, 02 Oct 2024 12:28:27 GMT
   Accept-Ranges: bytes
   Content-Length: 1952
   Cache-Control: max-age=15552000
   Expires: Tue, 01 Apr 2025 07:44:14 GMT
   Vary: Accept-Encoding
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   Content-Type: application/json
   
   {
   	"active": 1,
   	"pin": 515151,
           "min-build-ios": 18,
           "min-build-android": 13,
   	"product": {
   		"0": [
   			{
   	                  "fw-version": 16910354,
   	                  "fw-download": "https://www.almando.com/media/firmware/almando-decoder-V1_2_8_18.almpd",
           	          "mp-version": 16778246,
                   	  "mp-download": "https://www.almando.com/media/firmware/almando-multiplay_V1_0_4_6.almpd"
   			}
   		],
   		"3": [
   			{
   	                  "fw-version": 16908809,
   	                  "fw-download": "https://www.almando.com/media/firmware/almando-powerlinkswitch_V1_2_2_10.almpd"
   			}
   		],
     ......
    
   ```

   

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003154515515](https://s2.loli.net/2024/10/03/qUFN3tr5dwpGlJy.png)

   In total, we successfully downloaded **17 firmware** by exploiting the vulnerability. 


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