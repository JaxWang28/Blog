---
title: RustDesk 自建服务器
slug: RustDesk-Server-OSS
share: true
draft: true
date: 2024-11-02T18:11:54+08:00
tags: 
categories:
---

官方教程 RustDesk Server OSS - open source RustDesk Server. <br>
https://rustdesk.com/docs/en <br>


# RustDesk 框架


# 服务器端配置

1. 下载 rustdesk server
Github  https://github.com/rustdesk/rustdesk-server/releases <br>

2. 生成密钥对
运行 `./hbbs` 生成公钥私钥. <br>

3. Systemd Service 配置

```
# /etc/systemd/system/hbbs.service
[Unit]
Description=RustDesk Server Service
After=network.target

[Service]
Type=simple
User=root
# bin path
Restart=on-failure
RestartSec=5
WorkingDirectory=/home/jax/rustdesk
ExecStart=/home/jax/rustdesk/hbbs

[Install]
WantedBy=multi-user.target
```

```
# /etc/systemd/system/hbbr.service
[Unit]
Description=RustDesk Server Service
After=network.target

[Service]
Type=simple
User=root
# bin path
Restart=on-failure
RestartSec=5
WorkingDirectory=/home/jax/rustdesk
ExecStart=/home/jax/rustdesk/hbbr

[Install]
WantedBy=multi-user.target
```

```
service hbbr start
service hbbs start
service hbbs status
service hbbr status
```

允许自动启动 <br>
```
systemctl enable hbbr
systemctl enable hbbs
```

# 防火墙配置

默认情况下，hbbs 监听 `21115(tcp), 21116(tcp/udp), 21118(tcp)`，<br> 
hbbr 监听 `21117(tcp), 21119(tcp)` 。<br> 

21115: hbbs 用作 NAT 类型测试 <br>
21116/UDP: hbbs 用作 ID 注册与心跳服务 <br>
21116/TCP: hbbs 用作 TCP 打洞与连接服务 <br>
21117: hbbr 用作中继服务 <br>
21118/21119: 为了支持网页客户端。如果不需要网页客户端（21118，21119）支持，对应端口可以不开。