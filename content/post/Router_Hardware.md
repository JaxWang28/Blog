---
title: 路由器硬件组成
slug: router-hardware
share: true
draft: false
date: 2024-11-24T19:45:06+08:00
tags:
  - OpenWRT
  - Router
  - WiFi
categories:
---

本文以 `Redmi AX6000` 为例，介绍现代路由器中硬件组成，整理为以下部分：
* CPU
* RAM
* ROM Flash
* 交换芯片
* RF 芯片
* FEM 芯片

重要网站：<br>

**WikiDevi** is a user-editable database for computer hardware. https://wikidevi.wi-cat.ru/


# CPU
目前主流的路由器芯片为 **QualComm BraodCom MediaTek RealTek**。对于一款 CPU，尤其需要关注这些参数：**型号、架构、核心数、制程、频率**。  <br>

Redmi AX6000 CPU 参数：<br>
* 型号：MediaTek MT7986A
* 架构：ARM Cortex-A53
* 核心数：4 核
* 制程：12nm
* 频率：2.0 GHz

![](https://img.jaxwang.top/2024/11/593b4f15b720827f57cc27336a240a39.png)




# RAM
目前主流的 RAM 均为 Double Data Rate Synchronous Dynamic Random Access Memory，简称**DDR SDRAM**或**DDR**。关于 RAM 需要关注：**类型、型号、容量、频率**。 <br>

Redmi AX6000 RAM 参数：<br>
* 类型：DDR3
* 型号：K4A4G165WF-BCW 
* 容量：512 MB
* 频率：3200 MHz

![](https://img.jaxwang.top/2024/11/9372c151d96a21ba55fdf02e879fbde6.png)





# ROM Flash

Redmi AX6000 Flash 参数：<br>
* 型号：F50L1G41LB
* 容量：128MB

![](https://img.jaxwang.top/2024/11/4ad1efa4af7e3292989104efb5eea9e4.png)





# 交换芯片

关注点：吞吐量、包转发率、延迟、背板带宽、缓存大小、三层功能、VLAN、QoS、ACL、NAT、隧道协议、硬件加速能力。<br>


Redmi AX6000 交换芯片参数：<br>
* 型号：MT7531A



![](https://img.jaxwang.top/2024/11/1ec95b35864bf4766a59dbb32d482e9f.png)





# RF 芯片

## 2.4G RF
2.4G RF芯片型号是MT7976GN，通过”IQ“与MT7986A连接，支持4x4MIMO的Wi-Fi 6，在40MHz频宽和1024-QAM，最高速率1147Mbps

![](https://img.jaxwang.top/2024/11/3b5df8fd5e9715b3576a0b5267bf2fc0.png)

## 5G RF

Redmi AX6000 5G RF 芯片参数：<br>
* 型号：MT7976AN
* 4x4 MIMO WiFi 6
* 160MHz 1024-QAM 最高速率 4804Mbps

![](https://img.jaxwang.top/2024/11/fe09e99d90e078e89a5b4d14f08f1cd3.png)


**IQ 接口**：
红米 AX6000 的 MT7986A（主 SoC）和 MT7976AN（5G RF 芯片）之间通过 IQ 接口连接，这是 RF 系统中常见的设计。
**IQ 接口用于射频信号传输**
- IQ 接口允许主 SoC（MT7986A）将数字信号处理后的 **基带信号**（即 IQ 信号）传递给 RF 芯片（MT7976AN）。
- RF 芯片负责将 IQ 信号上变频为射频信号并通过天线发射，或将接收到的射频信号下变频为基带信号传回 SoC。




# FEM 芯片

**FEM 芯片**（**Front-End Module**，前端模块）是无线通信设备中的一种关键射频组件，集成了信号发送和接收路径中的多个前端功能，用于优化射频性能并简化设备设计。
### **FEM 芯片的基本概念**

FEM 芯片位于无线通信系统的射频前端（从天线到基带处理器之间的部分），它整合了与信号发射和接收相关的多个关键模块，例如功率放大器（PA）、低噪声放大器（LNA）、开关和滤波器。

**FEM 芯片的主要功能**
1. **信号发射路径**
    
    - **功率放大器（PA）**：
        - 将信号从基带处理器输出的低功率信号放大到足够高的功率，以便通过天线发射。
        - 需要在放大过程中尽量减少失真，确保信号质量。
2. **信号接收路径**
    
    - **低噪声放大器（LNA）**：
        - 在信号接收阶段对微弱的天线信号进行放大，同时尽量减少噪声。
        - 提高接收信号的信噪比（SNR）。
3. **开关功能**
    
    - 在发射和接收模式之间切换，确保信号不会互相干扰。
    - 开关通常与天线共享一个通道。
4. **滤波功能**
    
    - 使用滤波器滤除不需要的干扰信号或带外噪声。
    - 确保只有目标频段的信号被处理，优化系统性能。
5. **双工器或双工功能**
    
    - 支持全双工通信（同时发送和接收信号）。
    - 分离发射和接收信号的频段。


型号：RTC66568

![](https://img.jaxwang.top/2024/11/4fa8c496d793ee6fa77bfb0080a53224.png)







# 参考资料

Redmi AX6000 拆解 https://www.acwifi.net/19676.html <br>