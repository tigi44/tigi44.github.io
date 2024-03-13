---
title: "[NginX] NginX SSL"
excerpt: "Compare NginX SSL and NodeJS SSL"
description: "Compare NginX SSL and NodeJS SSL"
modified: 2020-06-22
categories: "NginX"
tags: [NginX, SSL, Server, HTTP, HTTPS]

toc: true
toc_sticky: true

header:
  teaser: /assets/images/teaser/nginx-teaser.png
---

# NginX Web Server
- Install : https://cofs.tistory.com/412
- nginx+nodejs : https://www.sitepoint.com/configuring-nginx-ssl-node-js/
https://velog.io/@jakeseo_me/Node에서-NGINX를-리버스-프록시로-사용하기-번역
- SSL : https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/
http://nginx.org/en/docs/http/configuring_https_servers.html
- redirect http to https : https://serversforhackers.com/c/redirect-http-to-https-nginx
- nginx 502 permission issue : https://cofs.tistory.com/411

# AS-IS : NodeJS 웹서버
- 테스트 : openssl s_client -connect 133.186.131.120:443
- 결과 : read:errno=0
```
CONNECTED(00000003)
depth=3 C = GB, ST = Greater Manchester, L = Salford, O = Comodo CA Limited, CN = AAA Certificate Services
verify return:1
depth=2 C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
verify return:1
depth=1 C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA
verify return:1
depth=0 C = KR, postalCode = 13487, ST = Gyeonggi, L = Seongnam, street = "16 Daewangpangyo-ro 645beon-gil, Bundang-gu", O = a Corporation, CN = *.a.com
verify return:1
---
Certificate chain
 0 s:/C=KR/postalCode=13487/ST=Gyeonggi/L=Seongnam/street=16 Daewangpangyo-ro 645beon-gil, Bundang-gu/O=a Corporation/CN=*.a.com
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
 1 s:/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
   i:/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority
 2 s:/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
 3 s:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIHLjCCBhagAwIBAgIQB9vtBVJ1/UaXwiBKUihXWTANBgkqhkiG9w0BAQsFADCB
lTELMAkGA1UEBhMCR0IxGzAZBgNVBAgTEkdyZWF0ZXIgTWFuY2hlc3RlcjEQMA4G
A1UEBxMHU2FsZm9yZDEYMBYGA1UEChMPU2VjdGlnbyBMaW1pdGVkMT0wOwYDVQQD
EzRTZWN0aWdvIFJTQSBPcmdhbml6YXRpb24gVmFsaWRhdGlvbiBTZWN1cmUgU2Vy
dmVyIENBMB4XDTIwMDEwMjAwMDAwMFoXDTIyMDQwMTIzNTk1OVowga8xCzAJBgNV
BAYTAktSMQ4wDAYDVQQREwUxMzQ4NzERMA8GA1UECBMIR3llb25nZ2kxETAPBgNV
BAcTCFNlb25nbmFtMTQwMgYDVQQJEysxNiBEYWV3YW5ncGFuZ3lvLXJvIDY0NWJl
b24tZ2lsLCBCdW5kYW5nLWd1MR4wHAYDVQQKExVOSE4gUEFZQ08gQ29ycG9yYXRp
b24xFDASBgNVBAMMCyoucGF5Y28uY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAqtJjIsvvpryFldokiFFbo76n9w02GhdloEwkmHaKt0pkqvTL08gD
HoEWbcPIAJchTYdpBvQn/dmIU8Ezs88bxWJbIdFNYu11SO6blZ5c4QoBiqfCJ6bo
2XDJiP9lRhiZY5VG9KrBg4UbSV+YjpIb0AYhoG2pXGc0dTuJJrYkjjbBlpnPPVwA
ddv1sgmdxHYI7gJadXqZPEQocnc+lWFR5nvwm3scat6U9ES8GjndPuK1f2fR/djA
IXc8X8CQ0eywdKf77dgr+BNgtvQ2L2T/2SYA6y3oMzDnmpfqxg6AznSl6bmLnyRM
cHsfk6oaZ8s3kLpfefw1+NB/7wiqxKXVbQIDAQABo4IDXDCCA1gwHwYDVR0jBBgw
FoAUF9nWJSdn+THCSUPZMDZEjGypT+swHQYDVR0OBBYEFEm4QTebTbPDhycudT3Z
5Bl9PHmpMA4GA1UdDwEB/wQEAwIFoDAMBgNVHRMBAf8EAjAAMB0GA1UdJQQWMBQG
CCsGAQUFBwMBBggrBgEFBQcDAjBKBgNVHSAEQzBBMDUGDCsGAQQBsjEBAgEDBDAl
MCMGCCsGAQUFBwIBFhdodHRwczovL3NlY3RpZ28uY29tL0NQUzAIBgZngQwBAgIw
WgYDVR0fBFMwUTBPoE2gS4ZJaHR0cDovL2NybC5zZWN0aWdvLmNvbS9TZWN0aWdv
UlNBT3JnYW5pemF0aW9uVmFsaWRhdGlvblNlY3VyZVNlcnZlckNBLmNybDCBigYI
KwYBBQUHAQEEfjB8MFUGCCsGAQUFBzAChklodHRwOi8vY3J0LnNlY3RpZ28uY29t
L1NlY3RpZ29SU0FPcmdhbml6YXRpb25WYWxpZGF0aW9uU2VjdXJlU2VydmVyQ0Eu
Y3J0MCMGCCsGAQUFBzABhhdodHRwOi8vb2NzcC5zZWN0aWdvLmNvbTAhBgNVHREE
GjAYggsqLnBheWNvLmNvbYIJcGF5Y28uY29tMIIBfwYKKwYBBAHWeQIEAgSCAW8E
ggFrAWkAdwBGpVXrdfqRIDC1oolp9PN9ESxBdL79SbiFq/L8cP5tRwAAAW9l1lRz
AAAEAwBIMEYCIQD/IBJi2FO1UWwxjuuk2OQeIPSnOKHo91rr+77eFn8fIgIhAMJA
uw8xc12QjFA4S1LS33l5HyQuEAfDQPwzCwRAra5WAHYAb1N2rDHwMRnYmQCkURX/
dxUcEdkCwQApBo2yCJo32RMAAAFvZdZVXAAABAMARzBFAiBHpsmDYoskdJivjIWn
oWmMRQKP/6K87wA4k1+LnEITwAIhAObyKCeKeOauQCJUzxx+lsjCLtlibselWpMr
a6C019ETAHYAIkVFB1lVJFaWP6Ev8fdthuAjJmOtwEt/XcaDXG7iDwIAAAFvZdZU
YAAABAMARzBFAiEA+e7wyjGXTBks1HGQ6FdZ7QjoV6LmFjnrN4wRbnQzC88CIDr4
ZMk39Q/qwlKjGFaEPahosW+XTVXKfr3iJtjDxILRMA0GCSqGSIb3DQEBCwUAA4IB
AQB/SBpGzSCiyn/TPI1UO1LUmD4fx7LHg/WMQOV1Shpv7pGqlQ3jLobLAu5zbCkr
uTzi4dOn69c6btHaYacN6r4diWKm+0O73d3yW6Wtg99wWJuHWFCYOLfAfGsNdBXR
LlRUr35Ipe7nGqxen/NilodOBhhgfdps1Q/wTWYPk/KKI5cGsTmVx6jFfsirPPSK
yWLCMDZtsSF6RmfYiCP7tpCxU7lPuKoSLfQivuwMspQ0a0d58V0hfM0n6DubpqfY
FwHq18BhTTUgJ+Hd+hg+X3fAuJUstPskCKLt2i6J2yxzvv4niXhwSfTAwO4tuwy+
INOD3mm8VZumCULE33Vc/aCe
-----END CERTIFICATE-----
subject=/C=KR/postalCode=13487/ST=Gyeonggi/L=Seongnam/street=16 Daewangpangyo-ro 645beon-gil, Bundang-gu/O=a Corporation/CN=*.a.com
issuer=/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
---
No client certificate CA names sent
Server Temp Key: ECDH, X25519, 253 bits
---
SSL handshake has read 6560 bytes and written 289 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 092E0BC2BD2115539C0EB5497E4ADAF738A6265C08FF297637385289A0CC5621
    Session-ID-ctx:
    Master-Key: 862CA9C4F10FD92E0E7288C861C7A0F627D9C48C24659E5EDFF1B6E4D2A5430CF56FB45754D3687CE82616E5AAAFB728
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 4a 57 4d 5b 54 7e 86 7c-0d 67 53 a3 5f 8e d1 03   JWM[T~.|.gS._...
    0010 - eb 0f a5 3e fe 0f a9 2d-00 44 a4 ab ec 37 13 33   ...>...-.D...7.3
    0020 - 14 25 2f b1 ca 53 b7 45-30 d0 5f 62 1c 6a d7 f8   .%/..S.E0._b.j..
    0030 - 62 38 6e 17 e9 19 04 db-5b 5f 4f 5d bd 90 20 5b   b8n.....[_O].. [
    0040 - 76 05 49 1c e7 b8 18 cc-9c 1d 82 95 ca d9 c5 bc   v.I.............
    0050 - 52 d8 a3 4c 0f d0 89 a4-dc 21 7c b3 94 b8 6c 11   R..L.....!|...l.
    0060 - da d7 e4 aa 56 60 f8 0f-77 98 21 3f 8e 65 9a db   ....V`..w.!?.e..
    0070 - f8 13 54 0b af cd 28 c5-c6 5c d8 c0 a7 f0 23 1e   ..T...(..\....#.
    0080 - 9d 49 64 f8 e9 56 9b 62-69 27 ca 7b 81 94 92 52   .Id..V.bi'.{...R
    0090 - 77 41 e1 1e c0 d1 87 7f-86 9c bc 46 7d 7e 80 64   wA.........F}~.d
    00a0 - e5 22 56 8c ed 82 58 ea-ce eb dc 19 ec fc 65 40   ."V...X.......e@
    00b0 - 59 4b f8 77 f2 f3 79 9c-5e f1 f6 14 97 42 5a 1f   YK.w..y.^....BZ.

    Start Time: 1592805884
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
read:errno=0
```

# TO-BE : Nginx 웹서버
- 테스트 : openssl s_client -connect 133.186.131.120:443
```
CONNECTED(00000003)
depth=3 C = GB, ST = Greater Manchester, L = Salford, O = Comodo CA Limited, CN = AAA Certificate Services
verify return:1
depth=2 C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
verify return:1
depth=1 C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Organization Validation Secure Server CA
verify return:1
depth=0 C = KR, postalCode = 13487, ST = Gyeonggi, L = Seongnam, street = "16 Daewangpangyo-ro 645beon-gil, Bundang-gu", O = a Corporation, CN = *.a.com
verify return:1
---
Certificate chain
 0 s:/C=KR/postalCode=13487/ST=Gyeonggi/L=Seongnam/street=16 Daewangpangyo-ro 645beon-gil, Bundang-gu/O=a Corporation/CN=*.a.com
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
 1 s:/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
   i:/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority
 2 s:/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
 3 s:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
   i:/C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIHLjCCBhagAwIBAgIQB9vtBVJ1/UaXwiBKUihXWTANBgkqhkiG9w0BAQsFADCB
lTELMAkGA1UEBhMCR0IxGzAZBgNVBAgTEkdyZWF0ZXIgTWFuY2hlc3RlcjEQMA4G
A1UEBxMHU2FsZm9yZDEYMBYGA1UEChMPU2VjdGlnbyBMaW1pdGVkMT0wOwYDVQQD
EzRTZWN0aWdvIFJTQSBPcmdhbml6YXRpb24gVmFsaWRhdGlvbiBTZWN1cmUgU2Vy
dmVyIENBMB4XDTIwMDEwMjAwMDAwMFoXDTIyMDQwMTIzNTk1OVowga8xCzAJBgNV
BAYTAktSMQ4wDAYDVQQREwUxMzQ4NzERMA8GA1UECBMIR3llb25nZ2kxETAPBgNV
BAcTCFNlb25nbmFtMTQwMgYDVQQJEysxNiBEYWV3YW5ncGFuZ3lvLXJvIDY0NWJl
b24tZ2lsLCBCdW5kYW5nLWd1MR4wHAYDVQQKExVOSE4gUEFZQ08gQ29ycG9yYXRp
b24xFDASBgNVBAMMCyoucGF5Y28uY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAqtJjIsvvpryFldokiFFbo76n9w02GhdloEwkmHaKt0pkqvTL08gD
HoEWbcPIAJchTYdpBvQn/dmIU8Ezs88bxWJbIdFNYu11SO6blZ5c4QoBiqfCJ6bo
2XDJiP9lRhiZY5VG9KrBg4UbSV+YjpIb0AYhoG2pXGc0dTuJJrYkjjbBlpnPPVwA
ddv1sgmdxHYI7gJadXqZPEQocnc+lWFR5nvwm3scat6U9ES8GjndPuK1f2fR/djA
IXc8X8CQ0eywdKf77dgr+BNgtvQ2L2T/2SYA6y3oMzDnmpfqxg6AznSl6bmLnyRM
cHsfk6oaZ8s3kLpfefw1+NB/7wiqxKXVbQIDAQABo4IDXDCCA1gwHwYDVR0jBBgw
FoAUF9nWJSdn+THCSUPZMDZEjGypT+swHQYDVR0OBBYEFEm4QTebTbPDhycudT3Z
5Bl9PHmpMA4GA1UdDwEB/wQEAwIFoDAMBgNVHRMBAf8EAjAAMB0GA1UdJQQWMBQG
CCsGAQUFBwMBBggrBgEFBQcDAjBKBgNVHSAEQzBBMDUGDCsGAQQBsjEBAgEDBDAl
MCMGCCsGAQUFBwIBFhdodHRwczovL3NlY3RpZ28uY29tL0NQUzAIBgZngQwBAgIw
WgYDVR0fBFMwUTBPoE2gS4ZJaHR0cDovL2NybC5zZWN0aWdvLmNvbS9TZWN0aWdv
UlNBT3JnYW5pemF0aW9uVmFsaWRhdGlvblNlY3VyZVNlcnZlckNBLmNybDCBigYI
KwYBBQUHAQEEfjB8MFUGCCsGAQUFBzAChklodHRwOi8vY3J0LnNlY3RpZ28uY29t
L1NlY3RpZ29SU0FPcmdhbml6YXRpb25WYWxpZGF0aW9uU2VjdXJlU2VydmVyQ0Eu
Y3J0MCMGCCsGAQUFBzABhhdodHRwOi8vb2NzcC5zZWN0aWdvLmNvbTAhBgNVHREE
GjAYggsqLnBheWNvLmNvbYIJcGF5Y28uY29tMIIBfwYKKwYBBAHWeQIEAgSCAW8E
ggFrAWkAdwBGpVXrdfqRIDC1oolp9PN9ESxBdL79SbiFq/L8cP5tRwAAAW9l1lRz
AAAEAwBIMEYCIQD/IBJi2FO1UWwxjuuk2OQeIPSnOKHo91rr+77eFn8fIgIhAMJA
uw8xc12QjFA4S1LS33l5HyQuEAfDQPwzCwRAra5WAHYAb1N2rDHwMRnYmQCkURX/
dxUcEdkCwQApBo2yCJo32RMAAAFvZdZVXAAABAMARzBFAiBHpsmDYoskdJivjIWn
oWmMRQKP/6K87wA4k1+LnEITwAIhAObyKCeKeOauQCJUzxx+lsjCLtlibselWpMr
a6C019ETAHYAIkVFB1lVJFaWP6Ev8fdthuAjJmOtwEt/XcaDXG7iDwIAAAFvZdZU
YAAABAMARzBFAiEA+e7wyjGXTBks1HGQ6FdZ7QjoV6LmFjnrN4wRbnQzC88CIDr4
ZMk39Q/qwlKjGFaEPahosW+XTVXKfr3iJtjDxILRMA0GCSqGSIb3DQEBCwUAA4IB
AQB/SBpGzSCiyn/TPI1UO1LUmD4fx7LHg/WMQOV1Shpv7pGqlQ3jLobLAu5zbCkr
uTzi4dOn69c6btHaYacN6r4diWKm+0O73d3yW6Wtg99wWJuHWFCYOLfAfGsNdBXR
LlRUr35Ipe7nGqxen/NilodOBhhgfdps1Q/wTWYPk/KKI5cGsTmVx6jFfsirPPSK
yWLCMDZtsSF6RmfYiCP7tpCxU7lPuKoSLfQivuwMspQ0a0d58V0hfM0n6DubpqfY
FwHq18BhTTUgJ+Hd+hg+X3fAuJUstPskCKLt2i6J2yxzvv4niXhwSfTAwO4tuwy+
INOD3mm8VZumCULE33Vc/aCe
-----END CERTIFICATE-----
subject=/C=KR/postalCode=13487/ST=Gyeonggi/L=Seongnam/street=16 Daewangpangyo-ro 645beon-gil, Bundang-gu/O=a Corporation/CN=*.a.com
issuer=/C=GB/ST=Greater Manchester/L=Salford/O=Sectigo Limited/CN=Sectigo RSA Organization Validation Secure Server CA
---
No client certificate CA names sent
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 6577 bytes and written 322 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: A6A496EC09E96C999A23E61A6180F9CFAEA9EA115BDC56BFFBDF413A9D5D356F
    Session-ID-ctx:
    Master-Key: AF2647D583076DB5CEF46A6F3F22CDB9A5501697961A24789EE22CF18861A440D6629F6EAE36E1F3E20DADBA36ABA098
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 99 b5 32 1f 42 34 bf 0b-d7 f6 5e ef 61 30 a6 93   ..2.B4....^.a0..
    0010 - ff c8 82 d3 5a 36 4a fd-53 3b 4f eb 0d 01 d8 0d   ....Z6J.S;O.....
    0020 - 73 42 17 5a ab 72 f0 85-1d eb 61 af da 75 7c 30   sB.Z.r....a..u|0
    0030 - 6e 8f 3c b0 37 ad b7 2b-f5 e6 eb 47 38 f1 be 8d   n.<.7..+...G8...
    0040 - 0d 09 d2 98 db be ab ab-03 7e 44 5d ed 89 67 3b   .........~D]..g;
    0050 - 24 f5 d9 37 95 c4 d0 89-8a 52 d5 1f e5 3a 21 4e   $..7.....R...:!N
    0060 - 2a d7 52 31 f6 f6 96 48-fe bd ab c2 48 44 2e 3b   *.R1...H....HD.;
    0070 - 5f 95 61 db 58 a1 71 a3-e6 40 ab af 0d 32 fc ed   _.a.X.q..@...2..
    0080 - 2f 37 dc 18 e3 ee a7 01-ef 80 35 c3 55 fc a3 53   /7........5.U..S
    0090 - 42 eb 85 59 f4 31 e2 3e-d3 14 56 30 d1 af 53 36   B..Y.1.>..V0..S6
    00a0 - 80 74 45 c7 98 a3 e3 8c-f7 3d d0 a7 fc 59 a0 c5   .tE......=...Y..

    Start Time: 1592895195
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
closed
```
