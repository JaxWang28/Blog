---
title: Github Pages 自定义域名
slug: GithugPages-Custom-Domain
draft: false
date: 2024-06-16T10:02:46+08:00
filename: Configuring_a_custom_domain_for_github_pages
---

https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site

要设置 `www` 或自定义子域（例如 `www.example.com` 或 `blog.example.com`），必须在存储库设置中添加你的域。 然后，通过 DNS 提供商配置 CNAME 记录。

![](https://img.jaxwang.top/2024/06/1d87b2526b600efe583e6f5ad5b943d2.png)

![](https://img.jaxwang.top/2024/06/db10a257fe7da679db3ec737c765df2a.png)
![](https://img.jaxwang.top/2024/06/bc63044c346688d59471efa374a4ba22.png)

![](https://img.jaxwang.top/2024/06/2ecb7ed66a8bc0d395b9bc5d93ae4fd2.png)

*关于域名解析记录的分类*

| 类型    | 用途                           |
| ----- | ---------------------------- |
| A     | 用于将域名解析到 IPv4 address        |
| AAAA  | 用于将域名解析到 IPv6 address        |
| CNAME | 用于将域名解析到另一个域名 Canonical Name |
