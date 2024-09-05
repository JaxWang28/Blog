---
title: EAP-Extensible Authentication Protocol
slug: extensible_authentication_rotocol
share: true
draft: false
date: 2024-09-05T23:53:25+08:00
tags: 
categories:
---
https://datatracker.ietf.org/doc/html/rfc3748<br>
Extensible Authentication Protocol 可扩展身份验证协议

EAP 被设计用来在网络中进行身份验证，但其不适用于 IP 层。
EAP 是一种**框架协议**，其本身并未规定如何进行身份验证，但其允许设计人员打造自己的 EAP 身份验证方式 EAP method。其在设计上能够运行于任何**链路层**，但其不适用于 IP 层。



```
 +-+-+-+-+-+-+-+-+-+-+-+-+  +-+-+-+-+-+-+-+-+-+-+-+-+
 |           |           |  |           |           |
 | EAP method| EAP method|  | EAP method| EAP method|
 | Type = X  | Type = Y  |  | Type = X  | Type = Y  |
 |       V   |           |  |       ^   |           |
 +-+-+-+-!-+-+-+-+-+-+-+-+  +-+-+-+-!-+-+-+-+-+-+-+-+
 |       !               |  |       !               |
 |  EAP  ! Peer layer    |  |  EAP  ! Auth. layer   |
 |       !               |  |       !               |
 +-+-+-+-!-+-+-+-+-+-+-+-+  +-+-+-+-!-+-+-+-+-+-+-+-+
 |       !               |  |       !               |
 |  EAP  ! layer         |  |  EAP  ! layer         |
 |       !               |  |       !               |
 +-+-+-+-!-+-+-+-+-+-+-+-+  +-+-+-+-!-+-+-+-+-+-+-+-+
 |       !               |  |       !               |
 | Lower ! layer         |  | Lower ! layer         |
 |       !               |  |       !               |
 +-+-+-+-!-+-+-+-+-+-+-+-+  +-+-+-+-!-+-+-+-+-+-+-+-+
		 !                          !
		 !   Peer                   ! Authenticator
		 +------------>-------------+
```




帧格式


```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Code      |  Identifier   |            Length             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Data ...
+-+-+-+-+
```