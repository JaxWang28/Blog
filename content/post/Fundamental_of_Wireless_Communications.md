---
title: 无线通信理论基础
slug: wifi-wireless-communication-fundamental
share: true
draft: false
date: 2024-09-17T22:04:00+08:00
tags: 
categories:
---

IEEE80211 需要无线通信理论的知识。<br>






# 射频信号

电磁波又称为电磁辐射，是能量的一种，是指同相振荡，且互相垂直的电场与磁场，在空间中以波的形式传递能量。
![](https://img.jaxwang.top/2024/09/0a9e56e6fa965417ff16a4b746206543.png)



电磁波伴随的电场方向，磁场方向，传播方向三者互相垂直。电磁波实际上分为电波和磁波，是二者的总称，但由于电场和磁场总是同时出现，同时消失，并相互转换，所以通常将二者合称为电磁波。**无线电波（射频信号）是电磁波的一种。**
![](https://img.jaxwang.top/2024/09/24787fe591e828a118b294ccdae3290d.webp)

电磁波有三大属性，即**振幅**（强度、光强）、**频率**（波长）和**波形**（频谱分布）。
![](https://img.jaxwang.top/2024/09/9739b604660b07643f03b09599b388fa.png)

射频信号始于发射机产生的交流电信号。

高频信号在通过各种物理介质时，衰减速度一般比低频信号要快。原因有二，自由空间路径损耗；频率越高信号穿越障碍物的能力越差。


损耗（衰减）
用于描述振幅或信号强度的降低。由于自由空间路径损耗的影响，射频信号也会随着距离的增大而减小。

自由空间路径损耗 FSPL
即使没有障碍物，射频信号也会随着距离的增大而减小。这是由于波的自然展宽而引起的信号减弱，也称为波束发散 beam divergence。<br>
FSPL 可以通过**6 dB**规则粗略估算，即**距离每翻一倍，振幅将减少 6 db**。<br>
![](https://img.jaxwang.top/2024/06/e503ef0b04a58dba1947b6602f0c6fb3.png)


多径
多径是一种传播现象，指的是信号沿两条或多条路径同时或相隔极短时间到达接收天线。<br>
**多径对遗留的 802.11a/b/g 无线接口具有破坏性影响，对采用 MIMO 天线分集与最大合并 MCR 信号处理技术的 802.11ac 无线接口具有建设性影响。**<br>
![](https://img.jaxwang.top/2024/06/3b0bac646faef2045c408753b7248cc6.png)









































**载波信号**
交流信号或直流信号本身不具备数据传输能力，但是如果信号发生很小的波动或变化，就能够将其解析出来。从而实现数据的收发，这种经过修改的信号称为载波信号。调整信号以产生载波信号的过程称为调制。<br>
通过对电波的振幅、频率、相位三个分量进行调制以产载h生载波信号。



**键控法**

收发器必须对信号进行处理，以便接收端可以正确区分 0 和 1。这种操作信号以表示多个数据的方法称为键控法 keying method。键控法将信号转化为载波信号。<br>


![](https://img.jaxwang.top/2024/09/df9899229525c7b762ecb922de0e45ad.png)

幅移键控通过改变信号的振幅或高度来表示二进制。<br>
频移键控通过改变信号的频率来表示二进制。**仅有某些遗留的 802.11 无线网络采用频移键控。采用频移键控实现高速传输的成本很高，该技术实用性差。**<br>
相移键控通过改变信号的相位来表示二进制。**相移键控技术广泛应用于 IEEE802.11-2016 标准中。**<br>
























































# OFDM 正交频分复用



![](https://img.jaxwang.top/2024/09/6ae5305da93cb3d6fb83fb5ddeebc917.png)
OFDM 技术将信道划分为独立、紧密且精确间隔的频率，这些频率被称为**子载波（副载波） subcarrier**，OFDM 副载波有时也被称为 OFDM 音调 tone。<br>



![](https://img.jaxwang.top/2024/09/f6bf8c009f6e230b20e3a1c53f7a0c4c.png)
子载波确定之后，就需要将子载波叠加发射出去。<br>

傅里叶变换认为任何一个波形都可以用无穷多个正弦波叠加而成。<br>

OFDM 通过反傅里叶变换 IFFT，将子载波叠加成时域信号发射出去。接收端通过傅里叶变换 FFT，分解子载波。

![](https://img.jaxwang.top/2024/09/56eb1cecaa52129665fccf00eac18d2d.png)



IEEE80211 中的 OFDM

IEEE80211-2016 规定 OFDM 使用 5GHz 频段传输数据，ERP-OFDM 使用 2.4GHz 传输数据。这两种技术在本质上并无区别。<br>
802.11a 和 802.11g 分别采用了 OFDM 和 ERP-OFDM。<br>

![](https://img.jaxwang.top/2024/07/ffc756c8bf95d134f33f769b0644f94e.png)<br>

20 MHz OFDM 信道由 64 个独立、紧密且精确间隔的频率构成，这些频率被称为**副载波 subcarrier**。<br>
副载波带宽：20 MHz / 64 = 312.5 kHz<br>
副载波间关系：副载波之间必须正交，不能相互影响。<br>
OFDM 符号时长：1 / 321.5 kHz = 3.2 us (根据副载波间正交性可以推到出该规则)<br>>
k
20 MHz 的 OFDM 信道副载波组成：<br>

| 数量  | 作用                                |
| --- | --------------------------------- |
| 4   | 导频载波 pilot carrier 发射机和接收机之间的动态校准 k|
| 12  | 充当保护频带 guard band 避免干扰            |
| 4   | 导频载波 pilot carrier 发射机和接收机之间的动态校准 |
| 48  | 用于传输调制数据                          |

**注意 40 MHz 带宽不是上述简单的 x2，802.11n 和 802.11ac 也使用OFDM技术，同样使用 64 个副载波构成的 20MHz 信道，但仅有 8 个副载波充当保护频带，52 个副载波用来传输调职数据，另有 4 个用作导频载波。**<br>


OFDM 调制：<br>
OFDM 采用 BPSK 和 QPSK 调制获得较低的速率，采用 16-QAM、64-QAM 和 256-QAM 获得较高的速率。<br>
*在后面会再学习*<br>




# 窄带与扩频

# 跳频扩频

# 直接序列扩频 


# OFDMA 正交频分多址




















# 信号变换




键控法



# 射频组件



# 功率单位与比较单位







# 射频数学计算


# 本底噪声


# 信噪比

# 信干噪比



# 接收信号强度指示









# 参考
https://zhuanlan.zhihu.com/p/693709981
【【小白也能看懂】-聊聊OFDM，子载波，傅里叶变换】 https://www.bilibili.com/video/BV12z411B7y9/?share_source=copy_web&vd_source=1975b8d28fe9daf8983e93a282352d93