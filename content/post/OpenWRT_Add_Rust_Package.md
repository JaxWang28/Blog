---
title: OpenWRT 构建 Rust Package
slug: openwrt_rust_package_build
share: true
draft: false
date: 2024-07-28T11:29:49+08:00
tags:
  - OpenWRT
  - Rust
categories:
---



# OpenWRT 构建系统搭建

1. 源码下载 
```
git clone  https://github.com/openwrt/openwrt.git
```


2. 版本更改
OpenWRT 从 [23.05.0](https://openwrt.org/releases/23.05/notes-23.05.0) 提供了 Rust Package Support
```
git checkout v23.05.4
# clean
make distclean
```


3. package 安装
```
./scripts/feeds update -a
./scripts/feeds install -a
```


```
ls ls feeds/packages/lang/rust/
```



4. 构建交叉编译环境配置文件
```
make menuconfig
勾选 Languages->rust
```


5. 编译交叉工具链

```
make toolchain/install
```

6. 调整 PATH 环境变量 目标无关工具和工具链被部署到 `staging_dir/host/` 和 `staging_dir/toolchain/` 目录。`staging_dir/host/bin` 可以找到相关的可执行文件。我们将其添加到 PATH 环境变量中方便我们直接使用。

```
export PATH=/home/buildbot/source/staging_dir/host/bin:$PATH
```



# 源码准备

1. 在 openwrt 的源码目录下执行
```
cargo new helloworld-rs
```




# 创建 package

1. 在 openwrt source code 下创建一个 feed 仓库，并创建一个 examples 类别。
```
mkdir -p mypackages/examples
```

2. 向该仓库中添加个一 helloworld-rs package
```
cd mypackages/examples
mkdir helloworld-rs
touch Makefile
```
每一个 package 都需要一个 Package Manifest 来描述这个 package，如从何处下载源码，如何构建源码。使用下面 Makefile 来实现，其中 SOURCE_DIR 就表明了获取源码的途径。
```
# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2023 Luca Barbato and Donald Hoskins

include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld-rs
PKG_VERSION:=1.0
PKG_RELEASE:=1

# Source settings (i.e. where to find the source codes)
# This is a custom variable, used below
SOURCE_DIR:=$(TOPDIR)/helloworld-rs

PKG_BUILD_DEPENDS:=rust/host
PKG_BUILD_PARALLEL:=1


include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/rust/rust-package.mk

define Package/helloworld-rs
  SECTION:=examples
  CATEGORY:=Examples
  TITLE:=Hello, World!
endef

# Package description; a more verbose description on what our package does
define Package/helloworld-rs/description
  A simple Rust "Hello, world!" -application.
endef

define Build/Prepare
                mkdir -p $(PKG_BUILD_DIR)
                cp -r $(SOURCE_DIR)/* $(PKG_BUILD_DIR)
                $(Build/Patch)
endef

$(eval $(call RustBinPackage,helloworld-rs))
$(eval $(call BuildPackage,helloworld-rs))
```



# 将 package 添加到 OpenWRT 构建系统

至此我们已经成功创建了一个 package，但是我们的构建系统还不知道这个 package 的存在。

1. 将 feed 仓库添加到配置文件中
```
touch feeds.conf
echo "src-link mypackages /openwrt_source_code/mypackages"  >> feeds.conf
```

2. 添加 mypackages 仓库中 hello world package 
```
./scripts/feeds update mypackages
./scripts/feeds install -a -p mypackages
```

 编译 package

1. 添加 package 到 firmware
```
make menuconfig
# selece the Examples->helloworld-rs
```

2. 编译
```
make package/helloworld-rs/compile
```

编译成功后可以在 `bin/packages/<arch>/mypackages` 找到我们的包文件。

# 总结

最终 OpenWRT 目录结构如下：<br>
```
mypackages/
└── examples
    └── helloworld-rs
        └── Makefile
helloworld-rs/
├── Cargo.lock
├── Cargo.toml
└── src
    └── main.rs
bin/packages/aarch64_cortex-a53/
└── mypackages
    └── helloworld-rs_1.0-1_aarch64_cortex-a53.ipk
feed.conf
```