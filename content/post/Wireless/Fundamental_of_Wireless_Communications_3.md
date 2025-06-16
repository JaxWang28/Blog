---
title: 无线通信理论基础（三）
slug: wifi-wireless-communication-fundamental-3
draft: true
date: 2024-06-29T21:32:20+08:00
tags:
  - IEEE80211
  - WiFi
  - Wireless
categories:
---

# 天线类型

天线不仅可以作为辐射器聚焦发射信号能量，还可以聚焦接收信号能量。<br>

## 全向天线

omnidirectional antenna<br>
![](https://img.jaxwang28.top/2024/06/8d67a716318a0407ad4e64bbccff2bf1.png)


## 半定向天线

semidirectional antenna<br>



## 强方向性天线

highly directional antenna<br>
![](https://img.jaxwang28.top/2024/06/2ab3b010ce0c5ac82c7f77835711222c.png)


# 天线分集
室内无线网络容易产生多径信号，为了补偿多径带来的影响，接入点等无线网络设备普遍采用天线分集 antenna diversity，又称为 spatial diversity，接入点使用两副或多副天线，并与接收机共同工作，以最大限度减少多径的不利影响。<br>
![](https://img.jaxwang28.top/2024/06/2477ab1897b0bffc445f4c20a0348d09.png)
接入点侦听到射频信号时，比较两副天线收到的信号，选择信号强的那副天线来接收数据帧。<br>
802.11n 之前的接口大多采用交换分集 switched diversity ，选择振幅最大的信号，忽略其他信号。<br>

# 多输入输出
多输入多输出 MIMO 是一种更为复杂的**天线分集**技术。<br>
多径会影响信号质量，因此传统的天线系统致力于消除多径带来的不利影响；而 MIMO 恰恰相反，它利用多径来改善信号质量，MIMO 使用多副天线同时收发数据，发射机能同时使用多个射频信号发射数据，接收机再将这些数据从信号中恢复。<br>
802.11n/ac 无线接口使用 MIMO。


