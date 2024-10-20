---
title: OpenWRT 构建个人 Package
slug: openwrt-new-package-build
draft: false
date: 2024-07-27T18:30:52+08:00
tags:
  - OpenWRT
categories:
---

最权威官方指导： https://openwrt.org/docs/guide-developer/helloworld/start<br>

# OpenWRT 编译系统准备

1. 源码下载 
```
git clone  https://github.com/openwrt/openwrt.git
```

2. 版本更改
```
git checkout v23.05.4
make distclean
```

3. 安装更新 feeds 避免后面出现问题
```
./scripts/feeds update -a
./scripts/feeds install -a
```

4. 配置交叉编译环境
```
make menuconfig
```

5. 编译交叉工具链
```
make toolchain/install
```


6. 调整 PATH 环境变量
目标无关工具和工具链被部署到 `staging_dir/host/` 和 `staging_dir/toolchain/` 目录。`staging_dir/host/bin` 可以找到相关的可执行文件。我们将其添加到 PATH 环境变量中方便我们直接使用。
```
export PATH=/home/buildbot/source/staging_dir/host/bin:$PATH
```




# 源码准备

1. 在 openwrt 的源码目录创建一个 `helloworld` 目录，并创建 `helloworld.c` 文件。
```
mkdir helloworld
touch helloworld.c
```


2. helloworld.c 如下
```
#include <stdio.h>
 
int main(void)
{
    printf("\nHello, world!\n\n");
	return 0;
}
```

# 创建 package

OpenWRT 高度依赖 **package** 的概念，几乎所有软件都是从 package 而来，包括交叉编译工具甚至是 kernel。<br>
而 **feed** 是一个包含多个软件包的集合。它类似于一个软件仓库，提供各种可安装的软件包和依赖项。它本质上就是一个仓库，可以是本地的，也可以是网络上的，例如 Github。


1. 在 openwrt source code 下创建一个 feed 仓库，并创建一个 examples 类别。
```
mkdir -p mypackages/examples
```

2. 向该仓库中添加个一 helloworld package
```
cd mypackages/examples
mkdir helloworld
touch Makefile
```
每一个 package 都需要一个 Package Manifest 来描述这个 package，如从何处下载源码，如何构建源码。使用下面 Makefile 来实现，其中 SOURCE_DIR 就表明了获取源码的途径。
```
include $(TOPDIR)/rules.mk

# Name, version and release number
# The name and version of your package are used to define the variable to point to the build directory of your package: $(PKG_BUILD_DIR)
PKG_NAME:=helloworld
PKG_VERSION:=1.0
PKG_RELEASE:=1

# Source settings (i.e. where to find the source codes)
# This is a custom variable, used below
SOURCE_DIR:=$(TOPDIR)/helloworld

include $(INCLUDE_DIR)/package.mk

# Package definition; instructs on how and where our package will appear in the overall configuration menu ('make menuconfig')
define Package/helloworld
  SECTION:=examples
  CATEGORY:=Examples
  TITLE:=Hello, World!
endef

# Package description; a more verbose description on what our package does
define Package/helloworld/description
  A simple "Hello, world!" -application.
endef

# Package preparation instructions; create the build directory and copy the source code. 
# The last command is necessary to ensure our preparation instructions remain compatible with the patching system.
define Build/Prepare
		mkdir -p $(PKG_BUILD_DIR)
		cp $(SOURCE_DIR)/* $(PKG_BUILD_DIR)
		$(Build/Patch)
endef

# Package build instructions; invoke the target-specific compiler to first compile the source file, and then to link the file into the final executable
define Build/Compile
		$(TARGET_CC) $(TARGET_CFLAGS) -o $(PKG_BUILD_DIR)/helloworld.o -c $(PKG_BUILD_DIR)/helloworld.c
		$(TARGET_CC) $(TARGET_LDFLAGS) -o $(PKG_BUILD_DIR)/$1 $(PKG_BUILD_DIR)/helloworld.o
endef

# Package install instructions; create a directory inside the package to hold our executable, and then copy the executable we built previously into the folder
define Package/helloworld/install
		$(INSTALL_DIR) $(1)/usr/bin
		$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/usr/bin
endef

# This command is always the last, it uses the definitions and variables we give above in order to get the job done
$(eval $(call BuildPackage,helloworld))
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



# 编译 package

1. 添加 package 到 firmware
```
make menuconfig
# selece the Examples->helloworld
```



2. 编译
```
make package/helloworld/compile
```

编译成功后可以在 `bin/packages/<arch>/mypackages` 找到我们的包文件。

# 总结

最终 OpenWRT 目录结构如下：<br>
```
mypackages/
└── examples
    └── helloworld
        └── Makefile
helloworld/
└── helloworld.c
bin/packages/aarch64_cortex-a53/
└── mypackages
    └── helloworld_1.0-1_aarch64_cortex-a53.ipk
feed.conf
```