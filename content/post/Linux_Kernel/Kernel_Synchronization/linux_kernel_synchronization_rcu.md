---
title: Linux Kernel 内核同步之 RCU
slug: linux-kernel-synchronization-rcu
share: true
draft: false
date: 2025-05-20T20:37:06+08:00
tags:
  - Kernel
  - KernelSynchronization
categories:
---

# Linux Kernel 内核同步之 RCU

尽管 `rwlock` 相对于 `spinlock` 做了很大的改进，`rwlock` 允许多个读者同时并发访问。但是进行写操作时，仅有一个写者具有访问权限，读者不可访问。`RCU` 进一步改善了这个问题，其允许多个读者和写者并发执行。而且 `RCU` 是不使用锁的，其对读者几乎没有限制，访问效率极高。本文简单概述这项技术。

## 一、RCU 概念理解

以下面的情景引出 RCU 。


1. 假设你写好一份作业，某一时刻同学A 找到你想借鉴一下你的作业，你把作业放在桌子上让同学A 借鉴。

2. 没过几分钟（同学A 还在借鉴 :) ），同学B 也来找到你，也想借鉴你一下你的作业，你指了指正在借鉴作业的同学A，说：“过去吧，你们两一起借鉴。”

3. 突然，你发现你的作业错写了一步，同学A 和同学B 正在借鉴（读操作）你的作业，此时你又没法修改（不可打断同学A 和同学B，也不能他们读的同时你修改）。怎么办？

4. 你以迅雷不及掩耳之势拿出一张A4纸（假设本次作业就是一张 A4纸），copy 原作页并开始改正。好巧不巧，此时同学C 又来借鉴你的作业了，而你此时还没改完，怎么办？那只好和同学A 同学B 一起去借鉴你的原作业。

5. 终于，你改正完了，此时你拥有了一份新的作业，原来那份就报废了（当然现在还不行，因为同学ABC 还在借鉴）。这时，同学D 也来了，想借鉴你的作业，这时你不能再让他借鉴你的旧作业了。你大方的指了指你的新作业。

6. 过了一会，同学ABC 都已经借鉴完你的旧作业了（虽然旧作业是有误的，但是你没责任提醒他们）。你发现已经没有同学借鉴你的旧作业了，你就可以把就作业销毁了。

以上就是一个 RCU 的过程。从上面我们发现，都是以**你**为中心，而不是你的作业。因此你可以拥有一份新的作业的同时暂时不销毁旧作业。你就像一个指针，指向一份作业。在拥有新作业的同时（malloc），而暂时不销毁旧作业（free）。这也就是为什么**RCU 只保护被动态分配并且通过指针引用的数据结构**。




## 二、RCU 5 core APIs


The core RCU API is quite small:

1. Updateer `rcu_assign_pointer`

```
  1 struct foo {
  2   int a;
  3   int b;
  4   int c;
  5 };
  6 struct foo *gp = NULL;
  7 
  8 /* . . . */
  9 
 10 p = kmalloc(sizeof(*p), GFP_KERNEL);
 11 p->a = 1;
 12 p->b = 2;
 13 p->c = 3;
 14 gp = p;
```

根据优化和内存屏障部分我们知道 11 12 13 14 行代码不一定按照我们想要的顺序执行，其很有可能先执行 14 行，再依次赋值。因此我们需要内存和优化屏障。Linux Kernel 已经封装了 `rcu_assign_pointer` 帮我们实现上述过程。其能保证下面代码先赋值 abc 结束后，再对 gp 赋值。
```
  1 p->a = 1;
  2 p->b = 2;
  3 p->c = 3;
  4 rcu_assign_pointer(gp, p);
```

2. `rcu_read_lock`
3. `rcu_read_unlock`
4. `rcu_dereference`

```
  1 p = gp;
  2 if (p != NULL) {
  3   do_something_with(p->a, p->b, p->c);
  4 }
```

同样，我们在读取端也要有类似的操作，同时读取端还要通过上锁，保证访问内容暂时不要被回收（旧作业）。

```
  1 rcu_read_lock();
  2 p = rcu_dereference(gp);
  3 if (p != NULL) {
  4   do_something_with(p->a, p->b, p->c);
  5 }
  6 rcu_read_unlock();
```

5. `synchronize_rcu` `call_rcu`
用于回收旧资源，启用 `synchronize_rcu` 是同步，其会 block 等待旧资源上所有的锁都释放，`call_rcu` 是异步。



## 三、RCU 基本流程

![](https://img.jaxwang.top/2025/05/f8d5c7c2e7482d64287db39491eaaa19.png)



## References

[1] What is RCU? -- “Read, Copy, Update” https://www.kernel.org/doc/html/next/RCU/whatisRCU.html

[2] What is RCU, Fundamentally? https://lwn.net/Articles/262464/