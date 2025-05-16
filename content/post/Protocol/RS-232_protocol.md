---
title: RS-232 串口通信协议
slug: rs232-protocol
share: true
draft: false
date: 2025-05-16T12:12:00+08:00
tags:
  - protocol
categories:
---

# RS-232 串口通信协议

串口通信（Serial Communication）是一种常用的全双工**系统间通信方式**。其使用 RS-232 标准。本文将从物理层和协议层讲解 RS-232 标准。

![](https://img.jaxwang.top/2025/05/8f404cbc844bb57a6d272703ad138a10.png)

## 一、物理层

RS-232 标准使用 9 根信号线，其接口被称为 COM 口（DB9 口）。

![](https://img.jaxwang.top/2025/05/a8aa058ba5200ee1e3184705e041e59a.png)

![](https://img.jaxwang.top/2025/05/010bcb3789abb61752b58e1c34a42168.png)

在目前的其它工业控制使用的串口通讯中，**一般只使用RXD、TXD以及GND三条信号线**， 直接传输数据信号，而RTS、CTS、DSR、DTR及DCD信号都被裁剪掉了。

常见的电子电路中常使用 TTL 的电平标准，使用 5V 表示逻辑1，使用 0V 表示逻辑0。但是为了提升远距离传输和抗干扰能力，RS-232 标准使用 -15V 表示逻辑1，+15V 表示逻辑0。因为控制器一般使用TTL电平标准，所以常常会使用MAX3232芯片对TTL及RS-232电平的信号进行互相转换。

|电平标准|描述|
| --- | --- |
|5V TTL|逻辑1: 2.4 ~ 5V<br>逻辑0: 0 ~ 0.5V|
|RS-232|逻辑1: -15 ~ -3V<br>逻辑0: +3 ~ +15V|

<center><img src="https://img.jaxwang.top/2025/05/5b401a051ccc7b9af23048dc20e26626.png" width="50%" height="50%"> </center>

## 二、协议层

RS-232 标准的数据包由发送端的 TXD 接口发送到接收端的 RXD 接口。其数据包由**起始位、数据位、校验位和停止位**组成。通信双方必须对数据包的格式约定一致才可正确通信。

![](https://img.jaxwang.top/2025/05/865e7a5f6476c480303a824543d8ad10.png)

* 空闲：空闲时一直处于高电位即逻辑1
* 起始位：数据包的起始信号由一个逻辑0的数据位表示
* 有效数据：有效数据的长度常被约定为5、6、7或8位长
* 数据检验：检验方法有奇校验(odd)、偶校验(even)、0校验(space)、1校验(mark)以及无校验(noparity)
* 停止位：数据包的停止信号可由0.5、1、1.5或2个逻辑1的数据位表示，只要双方约定一致即可

<center><img src="https://img.jaxwang.top/2025/05/3f09d2ad784bdaa1d3ad0034f6636a0d.png"  alt="串口软件配置界面" width="70%" height="70%"> </center>

由于 RS-232 是串口异步通讯，即通信过程中没有时钟信号（DB9接口中是没有时钟信号的），所以两个通讯设备之间需要约定好波特率，即每个码元的长度，以便对信号进行解码。常见的波特率为4800、9600、115200等。

## Ref
https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/USART.html 野火STM32

