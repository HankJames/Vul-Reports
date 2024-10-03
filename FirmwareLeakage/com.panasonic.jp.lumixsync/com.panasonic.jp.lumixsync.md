# Vulnerability Report - com.panasonic.jp.lumixsync- Incorrect Access Control

**Product:** `com.panasonic.jp.lumixsync`

**Vendor:** [Panasonic Holdings Corporation](https://av.jpn.support.panasonic.com/support/software/lumix_sync/index.html?r)

**Version:** 2.0.9

**Vulnerability Type:** Incorrect Access Control

**Severity:** High

## Description

The `com.panasonic.jp.lumixsync` , developed by **Panasonic Holdings Corporation**, suffers from an Incorrect Access Control vulnerability during the device firmware update process. The app uses HTTPS requests to download firmware updates. By reverse engineering the app, it was possible to identify the firmware download mechanism and the download link. Dynamic testing revealed that the vendor's firmware server lacks proper access control, leading to **firmware leakage**.

## Reproduction Steps

1. The `m8077ec`'s parameter value in the `run()`method provides the necessary URL and other information required for the HTTPS request. Static reverse engineering allows for the reconstruction of the complete request as follows:

   ![image-20241003192734872](https://s2.loli.net/2024/10/03/ktsnjvN1WoclV34.png)

2. Sending this URL request to the firmware server results in a direct response containing the latest firmware information as follows:

   > Request

   ```http
   GET https://av.jpn.support.panasonic.com/support/share/eww/com/software/lumix_sync/firm_list.xml HTTP/1.1
   User-Agent: Fiddler Everywhere
   Host: av.jpn.support.panasonic.com
   ```

   > Response:

   ```http
   HTTP/1.1 200 OK
   Server: Apache
   Last-Modified: Tue, 23 Jul 2024 01:06:40 GMT
   ETag: "625-61ddfc62cc800"
   Accept-Ranges: bytes
   Content-Length: 1573
   X-Frame-Options: SAMEORIGIN
   X-XSS-Protection: 1; mode=block
   X-Content-Type-Options: nosniff
   Content-Type: text/xml
   Expires: Thu, 03 Oct 2024 11:30:25 GMT
   Cache-Control: max-age=0, no-cache, no-store
   Pragma: no-cache
   Date: Thu, 03 Oct 2024 11:30:25 GMT
   Connection: keep-alive
   Set-Cookie: SRV=2210ae29-8db4-4299-b024-6a23967b7269; path=/
   
   <?xml version="1.0" encoding="utf-8"?>
   
   <firm_list version="1.0" date="20240723">
   
       <!--カメラボディ-->
       <body>
   
           <!--DC-S5-->
           <firm modelNumber="DC-S5" version="2.7" url="https://panasonic.jp/support/share/eww/com/software/lumix_sync/dsc/ff/s5___v27/firm_info.xml">
           </firm>
   
           <!--DC-GH6-->
           <firm modelNumber="DC-GH6_" version="3.0" url="https://panasonic.jp/support/share/eww/com/software/lumix_sync/dsc/fts/gh6__v30/firm_info.xml">
           </firm>
   ```

   Consequently, the unprotected firmware can be directly downloaded using this information.

   ![image-20241003193447189](https://s2.loli.net/2024/10/03/zpT1VoNE5HS2nrl.png)

   In total, we successfully downloaded **7 firmware** by exploiting the vulnerability.


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