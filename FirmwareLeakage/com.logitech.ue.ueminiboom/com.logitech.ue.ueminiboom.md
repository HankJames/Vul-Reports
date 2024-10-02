# Vulnerability Report - com.logitech.ue.ueminiboom- Incorrect Access Control

**Product:** `com.logitech.ue.ueminiboom` 

**Vendor:** [Logitech Europe S.A.](https://www.logitech.com/en-eu)

**Version:** 1.2.29

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.logitech.ue.ueminiboom` , developed by Logitech Europe S.A., suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `MANIFEST_URL` String value in the `AppSettings` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241002235909050](https://s2.loli.net/2024/10/02/j5Z7pQcsPznVJxF.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://files.logitech.com/audio/mobileApps/resources/boweryManifest.xml HTTP/1.1
   Host: files.logitech.com
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 66133
   Accept-Ranges: bytes
   Age: 182
   Alt-Svc: h3=":443"; ma=86400
   Cache-Control: max-age=300
   Content-Type: text/xml
   Date: Wed, 02 Oct 2024 15:58:11 GMT
   Etag: "10255-55a1a9b015e77"
   Expires: Wed, 02 Oct 2024 16:00:09 GMT
   Last-Modified: Tue, 26 Sep 2017 16:59:59 GMT
   Server: Apache
   Via: 1.1 044db435c889c784fb7699a7b74ad574.cloudfront.net (CloudFront)
   X-Amz-Cf-Id: ircpqaUOsKkJy6JKLbTPD6fqRv9VHvmKFZ8HqMj3mkRZ86bYRCbZ9w==
   X-Amz-Cf-Pop: HKG62-C2
   X-Cache: Hit from cloudfront
   
   <?xml version="1.0" encoding="UTF-8"?>
   
   <!-- Application properties for UE mobile apps -->
   <!-- "it's not just for Bowery anymore..." -->
   
   <appProperties version="201404021715">
   
     <!-- Orpheum offer causes app to selectively show a "Special Offer" button in certain locations -->
   <!--  <orpheumOffer offerURL="files.logitech.com/audio/mobileApps/resources/testOffer.php"
                   offerClickCount="2" />
   -->
   
     <!-- Triggers for firmware notification flag in app -->
     <firmwareList>
       <firmware device="Kora" hardwareRev="1.0.0" latestFirmware="7.2.145"
                 firmwareDownloadURL="files.logitech.com/audio/mobileApps/resources/firmware/kora_7.2.145_Release_Klassic_light.dfu"
   			  firmwareDetails="www.ultimateears.com/ueapp/ueboom2-update.html"/>
       <firmware device="Kora" hardwareRev="1.0.0" currentFirmware="default"
                 firmwareInfoURL="www.UltimateEars.com/boom-update" />
   
       <firmware device="Kora" hardwareRev="1.5.0" latestFirmware="8.2.145"
                 firmwareDownloadURL="files.logitech.com/audio/mobileApps/resources/firmware/kora_8.2.145_Release_light.dfu"
   			  firmwareDetails="www.ultimateears.com/ueapp/ueboom2-update.html"/>
       <firmware device="Kora" hardwareRev="1.5.0" currentFirmware="default"
                 firmwareInfoURL="www.UltimateEars.com/boom-update" />
                 ......
   ```
   
3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003000004447](https://s2.loli.net/2024/10/03/LO9vgX8tUQpRCyW.png)


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