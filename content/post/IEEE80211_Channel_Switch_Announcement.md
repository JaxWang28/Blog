---
title: "[IEEE80211] Channel Switch Announcement"
slug: ieee80211-vhannel-switch-announcement
share: true
draft: true
date: 2024-09-13T22:31:03+08:00
tags: 
categories:
---

# 0x01 简介
为了保持更好的连续性，AP 应该在其将要更换 Channel 时，通知连接在其上面的 Stas。


# 0x02 实现

```
  AP -------------channel 52------------- Client
  |
  |  will change to ch40
  V
  AP -------------channel 52------------- Client
  |   ---- Send new channel/bandwidth---->   |  stop tx prepare to change
  |                                          |
  |                                          |
  |  CSA count down to 1,                    |  CSA count down to 1,
  V  change to new channel                   V  change to new channel   
  AP -------------channel 40------------- Client


```

通过在 **Beacon 帧** 和 **Probe Response 帧** 中添加 **Channel Switch Announcement elements** 来实现此功能。
![](https://img.jaxwang.top/2024/09/9cfdc5379d31491d23d9c99252c49c59.png)
<center>ACS Element frame</center>





# 0x03 参考
IEEE80211-2016 9.4.2.19
