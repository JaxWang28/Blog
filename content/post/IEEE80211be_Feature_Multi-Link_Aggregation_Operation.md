---
title: "[IEEE80211be] MLO: Multi-Link Aggregation Operation"
slug: ieee80211be_mlo_multi-link_operation
share: true
draft: false
date: 2024-09-15T22:12:02+08:00
tags: 
categories:
---

# 0x01 What is MLO?

在 Wi-Fi6 中，STA 与 AP 之间只在 1 个频段上建立连接（2.4G、5G、6G）。这意味着其他频段没有被使用，这无疑是一种浪费。虽然现有的 Wi-Fi 技术可以在 2.4G 5G 6G 之间切换，但切换带来的延迟可达 100ms 。<br>

IEEE80211be Wi-Fi7 引入 MLO 多链路聚合技术，使 STA 与 AP 可以在多个频段上同时建立连接，改善数据传输。<br> 

![](https://img.jaxwang.top/2024/09/72ffdf6f9a6e7d2fca6f18df793211ca.png)

通过在多个频段建立连接，MLO 可以带来**更大的吞吐量、更低的延迟、更好的可靠性。**<br>


# 0x02 MLO 相关概念

![](https://img.jaxwang.top/2024/09/0ab4ca892433da3b57a775822c26d5c3.png)
多链路设备的一个射频单元有至少两个以上的射频链路链接到空口，但对于LLC层仅只有一个MAC地址。相比于单链路设备，在射频链路上增加了冗余。设备根据使用场景与空口状态，进行不同链路的切换与协同，来保障数据能够更高效、更快速、低延迟地进行传输。


**Link**
代表实际的连接。Link 有自己独立的 MAC 地址。

**MLD**
Multi-Link Device，支持 MLO Device。<br>
协议规定MLD的mac可以跟两个Link中的一个相同，也可以是另外不同的唯一地址。


# asynchronous and synchronous

同步多链路传输，MLD 在多条 link 上同步传输帧，起始时间对齐。<br>
异步多链路传输，MLD 在多条 link 上异步传输帧，起始时间不对齐。

![](https://img.jaxwang.top/2024/09/a094dc352d15db23469ab8eb358b470e.png)



![](https://img.jaxwang.top/2024/09/fdb1f9b0da76c5e0e671808954a29615.png)

采用异步多链路时，每条链路的传输独立，互不影响，`STR` 工作模式被提出。<br>
但是当每个 link 所在的 Radio 没有充分隔离时，其中一条 link 的传输势必会对另一条 link 产生干扰，这种干扰为 `in-device conxextence(IDC)` 干扰。为了解决这个问题，采用同步多链路的 `Non-STR(NSTR)` 工作模式被提出。


# Simultaneous Transmit & Receive (STR)

STR 允许在多条 link 独立工作，link 间互不干扰。当两条 link 分别进行 TX 和 RX 时，便产生了一种全双工的现象。

![](https://img.jaxwang.top/2024/09/49847998f28024848cc378c191c9b0d2.png)

STR 与 双频双并发 DBDC 的区别？

# Non-Simultaneous Transmit and Receive (NSTR)

NSTR 不允许在多条 link 独立工作，在同一时间所有 link 必须同时接收或发送。

![](https://img.jaxwang.top/2024/09/ec2c3f8966f9a4c02fdb4f9b59743922.png)

*其他细节？*







# Multi-Link Multi-Radio (MLMR)

上述两种模式被称为 Multi-Link Multi-Radio 模式，这这种模式下，link 是被静态分配的，而不可以动态切换。<br>




# Ehanced Multi-Link Signal-Radio (EMLSR)















# 0xff 参考





```
https://wlanprofessionals.com/exploring-key-features-of-multi-link-operation-mlo-in-wi-fi-7/
https://www.tp-link.com/us/blog/1067/what-is-wifi-7-s-multi-link-operation-mlo-/

https://zhuanlan.zhihu.com/p/387761464
https://www.youtube.com/watch?v=ohexy5VE170&ab_channel=WirelessLANProfessionals
```