---
title: x86 Linux 启动第二阶段 - protected mode
slug: x86-linux-kernet-boot-setup
share: true
draft: true
date: 2025-04-20T11:04:06+08:00
tags:
  - Linux
  - Kernel
  - x86
categories:
---
bootloader 在将 kernel 加载到 `0x100000` 后设置 `boot_params.hdr.code32_start` ，这个地址 `protected_mode` 内核的开始地址即 `0x100000`。

```
(gdb) x/32x 0x100000
0x100000:       0xfc    0xfa    0x8d    0xa6    0xe8    0x01    0x00    0x00
0x100008:       0xe8    0x00    0x00    0x00    0x00    0x5d    0x83    0xed

readelf -x .head.text arch/x86/boot/compressed/vmlinux
0x00000000 fcfa8da6 e8010000 e8000000 005d83ed .............]..
0x00000010 0d8d8510 f0c90089 40020f01 10b81800 ........@.......
```

# 第一张全局描述符表 GDT

```
SYM_DATA_START_LOCAL(gdt)
        .word   gdt_end - gdt - 1
        .long   0
        .word   0
        .quad   0x00cf9a000000ffff      /* __KERNEL32_CS */
        .quad   0x00af9a000000ffff      /* __KERNEL_CS */
        .quad   0x00cf92000000ffff      /* __KERNEL_DS */
        .quad   0x0080890000000000      /* TS descriptor */
        .quad   0x0000000000000000      /* TS continued */
SYM_DATA_END_LABEL(gdt, SYM_L_LOCAL, gdt_end)
```

![](https://img.jaxwang28.top/2025/04/441e0ef44d4aa8bef05b2ab190446907.png)

我们会发现每个段的基地址都是 `0x0`，这意味着在 Linux 下逻辑地址与线性地址是一致的。换而言之，**Linux 并没有使用 x86 提供的分段机制**。
*事实上，分段和分页在某种程度上有点多余，它们都能划分物理空间。*

# early 4G boot pagetable

![](https://img.jaxwang28.top/2025/04/2d2c22c6ff14db80a66ab00413d57861.png)