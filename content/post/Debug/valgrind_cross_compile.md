---
title: valgrind 交叉编译
slug: valgrind-cross-compile
share: true
draft: false
date: 2025-04-19T21:45:06+08:00
tags:
  - embeded
  - valgrind
  - debug
categories:
---


# 交叉编译

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

# 带调试信息的 libc

valgrind 需要使用带调试信息的 `libc`，否则会报 `cannot be set up` 错误。

`libc` 可以在交叉编译工具链中找到。
```
$ find . -name "*ld*.so"
./aarch64-linux-gnu/libc/lib/ld-2.25.so
$ file ./aarch64-linux-gnu/libc/lib/ld-
ld-2.25.so             ld-linux-aarch64.so.1
$ file ./aarch64-linux-gnu/libc/lib/ld-2.25.so
./aarch64-linux-gnu/libc/lib/ld-2.25.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, BuildID[sha1]=73b23c16ff9fe3eab046636cd9dd6db9d3309f27, with debug_info, not stripped
```

# 启动

将编译产生的工具目录和 `libc` 放入板子，启动 valgrind.

```
export VALGRIND_LIB=/libexec/valgrind && valgrind --tool=memcheck --leak-check=full /tmp/ld-2.28.so program
```

如果调试的 daemon 程序，可以在运行一段时间后通过 `SIGTERM` 信号终止 `valgrind` 获取结果。

# Ref.
https://www.cnblogs.com/yucloud/p/armbuild_valgrind3.html
https://valgrind.org/downloads/repository.html