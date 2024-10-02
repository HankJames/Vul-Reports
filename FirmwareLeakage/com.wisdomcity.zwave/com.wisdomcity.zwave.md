# Vulnerability Report - com.wisdomcity.zwave- Incorrect Access Control

**Product:** `com.wisdomcity.zwave` 

**Vendor:** [Plug n Play Camera](http://www.starvedia.com/)

**Version:** 1.1.0

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.wisdomcity.zwave` , developed by Plug n Play Camera, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `firmware_link` String value in the `BuildConfig` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003002132246](https://s2.loli.net/2024/10/03/3khIJoPQZfR8nDY.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET http://www.starvedia.com/AutoUpgrade/Httpversions.json HTTP/1.1
   Host: www.starvedia.com
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Connection: close
   Content-Length: 7219
   Accept-Ranges: bytes
   Content-Type: application/json
   Date: Wed, 02 Oct 2024 16:22:06 GMT
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

3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003002259920](https://s2.loli.net/2024/10/03/rMUdWRfCy8KBqPg.png)


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