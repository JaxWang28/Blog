---
title: 无线通信理论基础（一）
slug: wifi-wireless-communication-fundamental-1
filename: Fundamental_of_Wireless_Communications_1.md
share: false
draft: true
date: 2024-06-19T23:29:42+08:00
tags:
  - IEEE80211
  - Wireless
  - WiFi
categories:
---

学习 WiFi 需要一些基础无线通信理论。<br>
![](https://img.jaxwang28.top/2024/06/14b3606032cb762104032bbfb6f75b50.png)

# 射频信号
发射机产生的交流电信号为射频信号。

## 射频信号的特性
波长频率振幅这里就不过多讲述了。<br>
相位并不是单一射频信号的特性，其是一个**相对**概念，描述的是两个信号之间的关系。这和我们的月相是类似的。**相位对理解多径 multipath 至关重要**。

## 射频信号的行为
射频信号在空气中传播千变万化，射频的传播行为有以下几种关键的行为：吸收、反射、散射、折射、衍射、衰减、自由空间路径损耗、多径、增益。下面重点讲几种<br>

### 衰减
信号在空间中传输，受吸收、距离和多径的影响而衰减。

### 自由空间路径损耗
即使没有障碍吸收和多径的影响，电磁信号在传输中也会受物理定律而衰弱。射频信号在离开天线之后，能量会扩散到更大的区域，导致信号强度减弱。<br>
这里讲一下 6dB 规则，即距离每增加 1 倍，振幅将减少 6dB。如图中 2000 到 4000，增加 1 倍之后，振幅减少 6dB。<br>
![](https://img.jaxwang28.top/2024/06/e503ef0b04a58dba1947b6602f0c6fb3.png)<br>`

### 多径
由于信号会发生反射、散射、折射或衍射，导致同一信号沿多条路径传输，同时或相隔极短的时间到达接收天线。<br>
![](https://img.jaxwang28.top/2024/06/3b0bac646faef2045c408753b7248cc6.png)<br>
由于反射信号路径更长，因此一般晚于主信号到达天线，多条路径之间的时间差称为时延扩展 delay spread。<br>
多径造成遗留的 802.11a/b/g 无线网络性能下降，因为多径会降低接收信号的强度和质量。<br>
但是，802.11n/ac 无线接口采用 MIMO 天线分集与 MRC 信号处理技术，使多径转变为提升网络性能的有利条件。<br>

https://wirelesspi.com/carrier-phase-based-ranging-in-indoor-multipath-channels/

### 增益
增益又称为放大，描述振幅或信号强度的增加。<br>
增益分为有源增益和无源增益，收发器 transceiver 产生有源增益，收发器和天线之间使用放大器也会产生有源增益；使用天线聚焦射频信号产生无缘增益，如下图。<br>
![](https://img.jaxwang28.top/2024/06/6a4be86a98cfbf4c24a96a3cedad5b2a.png)