---
title: 使用 CloudFlare R2 搭建图床
slug: cloudflare-r2-image
filename: Set_Up_Image_Hosting_with_CloudFlare_R2.md
share: false
draft: false
date: 2024-06-15T10:02:46+08:00
tags: 
categories: 
---


# 为什么选择 Cloudflare R2

白嫖！白嫖！还是 TMD 白嫖！

R2 计费基于存储的数据总量及对数据执行的两类操作。无需支付任何出口费用。免费额度对于我们个人用户完全够用。*(不被盗刷的情下)*

![](https://img.jaxwang.top/2024/06/336ed2df579dd1a99322b4d4b748978d.png)



# 创建 Bucket

![](https://img.jaxwang.top/2024/06/f0c424e0420f8dbe2b1ae5c73fab0c94.png)

![](https://img.jaxwang.top/2024/06/e277b0bd1504f73ac73a2697b078b0c8.png)



# 配置 Bucket

桶创建完毕之后，我们需要配置 Bucket，主要是添加自定义域名和创建令牌。前者用来公共访问我们 Bucket 中的文件，后者用来帮助我们上传到 Bucket。

配置自定义域名：设置 -> 公开访问 -> 连接域

![](https://img.jaxwang.top/2024/06/2e8d901c72db8e2c86924dfeea28bdcb.png)

![](https://img.jaxwang.top/2024/06/cf6abf7713d9dcb04535dfbdefdaf54e.png)

这里有 3 点要讲的：

* 自定义是用来公开访问我们的桶的
* 我们可以购买域名，然后在 Cloudflare 中注册管理
* 注册管理域名之后，可以在域中添加已经注册的域或子域

*至于如何购买域名，在 CloudFlare 中注册可以 google it，或许我后面会出一期教程*



创建令牌：管理令牌 -> 创建令牌

![](https://img.jaxwang.top/2024/06/fc7545157f24d288c12ef5692699ebf4.png)

![](https://img.jaxwang.top/2024/06/9ace4aab9389a9f72eae1bf0e29a68a2.png)

![](https://img.jaxwang.top/2024/06/979c555739fffa1262bd09f0471ff1c4.png)

至此，令牌创建完毕，不要急着关闭此页面。



# 配置 PicGo

PicGo 工具主要是用来帮助我们将图片上传到 R2 中。

首先安装上插件 S3。

![](https://img.jaxwang.top/2024/06/902490696ab8dc359f92079522118f73.png)

![](https://img.jaxwang.top/2024/06/de2b251817590116a3d883a9d0876c6c.png)
