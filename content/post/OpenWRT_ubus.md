---
title: OpenWRT ubus 框架
slug: OpenWRT-ubus
share: true
draft: false
date: 2024-07-28T20:47:05+08:00
tags:
  - OpenWRT
categories:
---

https://openwrt.org/docs/techref/ubus <br>
https://hackmd.io/@rYMqzC-9Rxy0Isn3zClURg/H1BY98bRw <br>

一直在使用 ubus，系统性学习一下，并准备用 rust 重构一把。


# ubus 概述

ubus 是一个 OpenWRT 中的 RPC 工具，其主要目的就是提供系统级进程间通信。<br>
其核心是 ubusd daemon，提供接口给任何其他 daemon 程序进行注册或发送信息。该接口使用 unix socket 实现，并使用 TLV 格式消息。<br>
ubus 提供了 3 种工具：
* libubus
* command-line tool ubus
* ubus lua module


# ubus 通信模型

先看一下我们常见的 client-server 模型。<br>
![](https://img.jaxwang.top/2024/07/c22c0e7464b519259d48e4ebc919159c.png)


假设有 n 个程序，那么进程间通信的总 IPC 连接数将达到 n 的累加，这是非常惊人的数量。<br>

使用 ubus IPC 模型。<br>
![](https://img.jaxwang.top/2024/07/337b7ddfd0334542f502db331c62f034.png)

**ubus daemon**：作为代理人，负责转发注册。<br>
**ubus server object**：一些软件或守护进程提供接口，供其他软件调用。Server object 以及其注册的方法可以被 client 调用。<br>
**ubus client object**：调用者。<br>


# ubus 工作模式



## Invoke
一对一的通信模式。
![](https://img.jaxwang.top/2024/07/7f0b542ed8e100a5d92fe23e8849761f.png)





![](https://img.jaxwang.top/2024/07/a9062b48edd65822cecef2a0950fb7ab.png)




## Subscribe/notify

一对多的通信模式，以 object 进行分组。<br>
![](https://img.jaxwang.top/2024/07/b3b08c141488348f69b93a602dd9a366.png)

![](https://img.jaxwang.top/2024/07/698a0d6cab87565293109af0c0863268.png)




## Event Boardcast

多对多的通信模式，通过 event 进行分组。<br>
![](https://img.jaxwang.top/2024/07/541d43cc5eaa8755644da527fd7a9511.png)


![](https://img.jaxwang.top/2024/07/82fc6a4e965763f04175404adbd87d54.png)


