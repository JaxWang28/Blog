---
title: x86 Linux 启动第一阶段 - setup
slug: x86-linux-kernet-boot-setup
share: true
draft: true
date: 2025-04-19T20:45:06+08:00
tags:
  - Linux
  - Kernel
  - x86
categories:
---

```
linux-6.12.4
QEMU emulator version 8.2.2 (Debian 1:8.2.2+ds-0ubuntu1.6)
```

# 魔数 MZ

```shell
readelf -x .bstext  arch/x86/boot/setup.elf
Hex dump of section '.bstext':
0x00000000 4d5a0000 00000000 00000000 00000000 MZ..............
```

```
xxd arch/x86/boot/setup.bin
00000000: 4d5a 0000 0000 0000 0000 0000 0000 0000  MZ..............
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

```
(gdb) hb *0x10200
(gdb) c
(gdb) x/2c 0x10000
0x10000 <exception_stacks+16384>:       77 'M'  90 'Z'
```


# setup 过程函数跳转
![](https://img.jaxwang28.top/2025/04/1ea16f28231769c5f553393cfd522178.png)

# _start

bootloader 将 `setup.bin` 加载到 `0x10000` 地址处后，将 `CS` 置为 `0x1020` ，`IP` 置为 `0x0`。通过计算可以得知地址为 `0x10200`。

```
nm arch/x86/boot/setup.elf  | grep 200
00000200 a falign
00000200 R _start
```

``` 
arch/x86/boot/header.S
        # offset 512, entry point

        .globl  _start
_start:
                # Explicitly enter this as bytes, or the assembler
                # tries to generate a 3-byte jump here, which causes
                # everything else to push off to the wrong offset.
                .byte   0xeb            # short (2-byte) jump
                .byte   start_of_setup-1f
1:
// start_of_setup -1f is offset 
// 0xeb is short jump, 0xeb <offset>
```

通过源码可知，进入 `0x10200` 后，跳转到 `start_of_setup` 处。


# start_of_setup

* 初始化寄存器状态
* 准备堆栈
* 跳转到 `main`


# main

* 各种方面的初始化


# go_to_protected_mode

* 为 `protected_mode` 做准备


# protected_mode_jump

* 通过汇编跳转到 `protected_mode`

```
jmpl    *%eax                   # Jump to the 32-bit entrypoint

// 这里的 eax 是 C 语言调用的第一个参数，在 pm.c 中可以找到参数为 boot_param.hdr.code32_start
```
