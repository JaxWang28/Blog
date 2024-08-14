---
title: WSL2 Kernel 升级更换
slug: wsl2_kernel_update
share: true
draft: false
date: 2024-08-14T11:04:51+08:00
tags:
  - WSL
  - Linux
  - Kernel
categories:
---

**! Warning**<br>
If you have multiple distros installed, they will all use the same kernel.<br>
WSL2 中不同发行版使用的是同一个内核。<br>

# 0x01 准备

1. 源码
```
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
```

2. toolchain 
```
sudo apt update && sudo apt install build-essential flex bison libssl-dev libelf-dev bc python3 pahole
```

3. 查看源码分支选择版本
```
# 查看所有版本
git tag 

# 选择目前最新版本
git checkout linux-msft-wsl-6.6.36.3
```

# 0x02 构建


```
make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl
```

```
make -j$(nproc) KCONFIG_CONFIG=Microsoft/config-wsl
```

# 0x03 安装

```
sudo make modules_install headers_install
```

```
cp arch/x86/boot/bzImage /mnt/c/User/<username>/bzImage
```

 创建 `%USERPROFILE%\.wslconfig` 并添加下面内容
```
[wsl2]
kernel=C:\\User\\<username>\\bzImage
```

# 0x04 检验

```
wsl --shutdown
```


```
uname -r
```

# 0x05 参考
https://learn.microsoft.com/en-us/community/content/wsl-user-msft-kernel-v6
