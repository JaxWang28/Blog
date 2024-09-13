---
title: EAP-Extensible Authentication Protocol
slug: extensible_authentication_rotocol
share: true
draft: false
date: 2024-09-05T23:53:25+08:00
tags: 
categories:
---
# 0x00 EAP 框架协议

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



# 0x01 封包格式

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Code      |  Identifier   |            Length             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Data ...
+-+-+-+-+
```


Code

| 1   | Request  |
| --- | -------- |
| 2   | Response |
| 3   | Success  |
| 4   | Failure  |

Identifier
标识符，重传时使用相同的标识符

Length
EAP 总长度，包括 code Identifier Length Data

Data


# 0x03 请求与响应


```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Code      |  Identifier   |            Length             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |  Type-Data ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
```

Type
This field indicates the Type of Request or Response.

Type-Data
The Type-Data field varies with the Type of Request and the associated Response.

Initial EAP Request/Response Types
 1       Identity
 2       Notification
 3       Nak (Response only)
 4       MD5-Challenge
 5       One Time Password (OTP)
 6       Generic Token Card (GTC)
254       Expanded Types
255       Experimental use


# 0xFF 参考文献
https://datatracker.ietf.org/doc/html/rfc3748

