---
title: Valgrind 基础介绍与框架概览
slug: valgrind-intro-framework-overview
share: true
draft: false
date: 2025-04-19T21:45:06+08:00
tags:
  - embeded
  - valgrind
  - debug
categories:
---


# Valgrind 基础介绍与框架概览

Valgrind是一款用于内存调试、内存泄漏检测以及性能分析的软件开发工具，本文简单分析 Valgrind 基本原理及其框架组成，并记录其在嵌入式设备上的移植与调试过程。

<center>
<img src="https://img.jaxwang.top/2025/05/ca0ba156ff8f0203ba14b1e713ea4318.png" width="50%" height="50%">
</center>

## 一、插桩技术概述

   [插桩](https://en.wikipedia.org/wiki/Instrumentation_(computer_programming))（instrumentation）技术是指**在程序源码或二进制代码中插入监测代码**实现对程序的监视。达到记录函数进入和退出、监控内存读写、统计执行时间、跟踪异常等。

插桩在实现上可分为两类：
1. 静态插桩 Static Instrumentation：在程序编译之前或编译期间插入代码，即在**源代码或中间代码**中插入检测代码，最终编译成带有插桩的可执行文件。
2. 动态插桩 Dynamic Binary Instrumentation DBI：在程序运行中进行插桩，其操作对象为**机器码**。动态二进制插桩工具主要包含 Pin，Valgrind，DynamoRIO 等。

## 二、Valgrind Framework

Valgrind 本质上是一套核心框架 framework，其提供了核心 Core 及 APIs ，在此框架基础上开发出 7 个工具：

<center>
<img src="https://img.jaxwang.top/2025/05/50281872ddcb9927d56a85162f9c62f2.png" width="50%" height="50%">
</center>


|   工具  | 描述    |
| --- | --- |
|Memcheck |内存错误检测，用于检测内存泄漏、越界访问、使用未初始化内存等内存相关错误|
|Helgrind |线程错误检测，检测数据竞争（race conditions）和死锁|
| DRD| 线程错误检测，使用与 Helgrind 不同技术|
|Cachegrind |缓存和分支预测分析|
| Callgrind |基于 Cachegrind，但额外生成调用图，分析函数调用及其性能影响|
|Massif|堆内存使用分析工具，帮助找出内存使用高峰及其来源|
|DHAT |分析堆内存分配和释放的详细情况|

## 三、嵌入式平台调试 memory leak

1. 指定 `toolchains`

```
export GCC_PATH=/your_path_for_linaro/bin
export CC=${GCC_PATH}/aarch64-linux-gnu-gcc
export LD=${GCC_PATH}/aarch64-linux-gnu-ld
export AR=${GCC_PATH}/aarch64-linux-gnu-ar
```

2. 编译配置

```
cd valgrind
./autogen.sh
./configure --prefix=`pwd`/Inst --host=aarch64-unknown-linux --enable-only64bit
```

3. 编译安装

```
make -j4 install
```

4. 准备带有debug info 的 libc

valgrind 需要使用带调试信息的 `libc`，否则会报 `cannot be set up` 错误，在 `libc` 可以在交叉编译工具链中找到。

```
$ find . -name "*ld*.so"
./aarch64-linux-gnu/libc/lib/ld-2.25.so
$ file ./aarch64-linux-gnu/libc/lib/ld-
ld-2.25.so             ld-linux-aarch64.so.1
$ file ./aarch64-linux-gnu/libc/lib/ld-2.25.so
./aarch64-linux-gnu/libc/lib/ld-2.25.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, BuildID[sha1]=73b23c16ff9fe3eab046636cd9dd6db9d3309f27, with debug_info, not stripped
```

5. 开始检测

将编译产生的工具目录和 `libc` 放入板子，启动 valgrind.

```
export VALGRIND_LIB=/libexec/valgrind && valgrind --tool=memcheck --leak-check=full /tmp/ld-2.28.so program
```

如果调试的 daemon 程序，可以在运行一段时间后通过 `SIGTERM` 信号终止 `valgrind` 获取结果。


## References

valgrind.org https://valgrind.org/

https://www.cnblogs.com/yucloud/p/armbuild_valgrind3.html

https://valgrind.org/downloads/repository.html

论文 模糊测试中的静态插桩技术 https://crad.ict.ac.cn/cn/article/pdf/preview/10.7544/issn1000-1239.202220883.pdf

论文 Valgrind: A Framework for Heavyweight Dynamic Binary Instrumentation https://valgrind.org/docs/valgrind2007.pdf

https://zhuanlan.zhihu.com/p/382100526

https://robotchaox.github.io/linux-tool/valgrind%E5%86%85%E5%AD%98%E6%A3%80%E6%B5%8B.html

https://www.semanticscholar.org/paper/Runtime-Overhead-Reduction-in-Automated-Parallel-Hoshi-Ootsu/3035e38a830e14daf02d55c13037f101e55b0246