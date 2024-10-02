# Vulnerability Report - com.home.shelly- Incorrect Access Control

**Product:** `com.home.shelly` 

**Vendor:** [Shelly](https://www.shelly.com/)

**Version:** 1.0.4

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.home.shelly` , developed by Shelly, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

> Although the vendor has discontinued the use of the application, the URL for obtaining firmware has not been deprecated.

## Reproduction Steps

1. The `checkFirmware(JSONObject)` method in the `BackgroundFCMHandler` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241002220234191](https://s2.loli.net/2024/10/02/Mg3u1S64IcCW97m.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://api.shelly.cloud/files/firmware HTTP/1.1
   Host: api.shelly.cloud
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Server: nginx/1.18.0 (Ubuntu)
   Date: Wed, 02 Oct 2024 14:04:10 GMT
   Content-Type: application/json
   Transfer-Encoding: chunked
   Connection: keep-alive
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Methods: GET,POST,OPTIONS
   Access-Control-Allow-Headers: X-CustomHeader,Keep-Alive,User-Agent,Cache-Control,Content-Type,Authorization
   Access-Control-Expose-Headers: X-CustomHeader,Keep-Alive,User-Agent,Cache-Control,Content-Type,Authorization
   
   1eab
   {"isok":true,"data":{"SHPLG-1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-1.zip","version":"20210115-103101\/v1.9.4@e2732e05"},"SHPLG-S":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-S.zip","version":"20230913-113421\/v1.14.0-gcb84623","beta_url":"http:\/\/firmware.shelly.cloud\/gen1\/rc\/SHPLG-S.zip","beta_ver":"20231107-164219\/v1.14.1-rc1-g0617c15"},"SHPLG-IT1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-IT1.zip","version":"20211109-130408\/v1.11.7-g682a0db"},"SHPLG-UK1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-UK1.zip","version":"20211109-130512\/v1.11.7-g682a0db"},"SHPLG-AU1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-AU1.zip","version":"20211109-130618\/v1.11.7-g682a0db"},"SHPLG-U1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG-U1.zip","version":"20230913-113516\/v1.14.0-gcb84623","beta_url":"http:\/\/firmware.shelly.cloud\/gen1\/rc\/SHPLG-U1.zip","beta_ver":"20231107-164310\/v1.14.1-rc1-g0617c15"},"SHPLG2-1":{"url":"http:\/\/firmware.shelly.cloud\/gen1\/SHPLG2-1.zip","version":"20230913-113610\/v1.14.0-gcb84623","beta_url":"http:\/\/firmware.shelly.cloud\/gen1\/rc\/SHPLG2-1.zip","beta_ver":"20231107-164401\/v1.14.1-rc1-g0617c15"},
   ......
   ```

3. Consequently, the unprotected firmware can be directly downloaded using this information.
   ![image-20241002220700908](https://s2.loli.net/2024/10/02/P7avbuYGI2osA56.png)

   ![image-20241002220723797](https://s2.loli.net/2024/10/02/rIWzDyNtdgv1Mme.png)


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