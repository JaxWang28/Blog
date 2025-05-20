---
title: Linux Kernel 内核同步之自旋锁
slug: linux-kernel-synchronization-spinlock
share: true
draft: false
date: 2025-05-19T22:37:06+08:00
tags:
  - Kernel
  - KernelSynchronization
categories:
---

# Linux Kernel 内核同步之自旋锁

加锁（locking）是一种广泛应用的同步技术。当要访问**共享数据结构**或进入**临界区**时，为自己获得一把“锁”，离开时再释放锁。在获得锁到释放锁之间的之短时间内是禁止其他 CPU 访问的。

```
spin_lock(&lock);
...
spin_unlock(&lock);
```

不同系统架构的自旋锁实现是不同的：
1. 不支持内核抢占的单处理器：自旋锁的实现是空的
2. 支持内核抢占的单处理器：自旋锁的实现仅仅是禁止或启用**内核抢占**。参考下面例子：
	1. 线程 A 获得锁
	2. 线程 B 抢占内核获得执行权
	3. 线程 B 获取相同锁发现已被锁
	4. . 线程 B 的优先级高于线程 A，死锁
3. 支持内核抢占的多处理器：需要完整实现自旋锁

`spin_lock` 是不会禁用**本地中断**的，但是提供了 `spin_lock_irq` ，其不仅会禁用内核抢占并且会禁用本地中断。当需要上锁的共享资源会在中断处理中被访问，我们需要使用禁止本地中断的锁，原因如下：
1. 线程 A 获得锁
2. 触发中断，中断处理中要获得相同锁，死锁

`spin_lock_irq()` 在任何情况下都是安全的，但是没有 `spin_lock()` 快。[3]


|   代数  |   名称  | 内核版本 | 公平性 | 高效性 | 接口兼容性 |
| --- | --- | --- | --- | --- | --- |
|第一代|原始自旋锁|1.0 - 2.6.24| 不公平 | 不高效 | 兼容 |
|第二代|ticket spinlock|2.6.25 - 4.1x| 公平 | 不高效 | 兼容 |
|第三代|MCS spinlock |3.15 - now | 公平 | 高效 | 不兼容 |
|第四代|queued spinlock|4.2 - now| 公平 | 高效 | 兼容 |

自旋锁首次提出之后又经历了三次迭代，下面对其简述：
1. 第一代原始自旋锁，对于等待锁的 CPU 只是无脑的“盲等”（尽管等待锁时时没有禁用内核抢占，可以调度执行其他任务），其不具有公平性，即没有先后顺序。
2. 第二代 ticket spinlock，其具有公平性，但是不高效。

对于 ARM64 已经不支持第一代和第二代 spinlock , 本文着重讲解 MCS spinlock 和 queued spinlock。

```
commit c11090474d70590170cf5fa6afe85864ab494b37
Author: Will Deacon <will@kernel.org>
Date:   Tue Mar 13 20:45:45 2018 +0000

    arm64: locking: Replace ticket lock implementation with qspinlock

    It's fair to say that our ticket lock has served us well over time, but
    it's time to bite the bullet and start using the generic qspinlock code
    so we can make use of explicit MCS queuing and potentially better PV
```

## 一、spinlock APIs

|API     |    功能 |
| --- | --- |
|   spin_lock(&lock)  |  获取自旋锁   |
|spin_unlock(&lock)|释放自旋锁|
|spin_trylock(&lock)|尝试获取锁，成功返回 true|
|spin_is_locked(&lock)|检查锁是否被持有|
|spin_unlock_wait(&lock)|等待锁释放，不加锁|
|spin_lock_irq(&lock)|获取锁 + 禁用本地 CPU 的中断|
|spin_unlock_irq(&lock)|解锁 + 恢复中断|
|spin_lock_irqsave(&lock, flags)|获取锁 + 保存当前中断状态 + 禁用中断|
|spin_unlock_irqrestore(&lock, flags)|解锁 + 恢复之前的中断状态|
|spin_lock_bh(&lock)|取锁 + 禁用软中断 |


## 二、MCS spinlock
ticket spinlock 完美解决了公平问题，但是存在一个问题，就是当锁竞争比较激烈的时候，大家都在自旋同一个变量，造成 **多核频繁竞争同一个缓存行**，引发 **缓存颠簸**，进而拖慢整个系统的同步效率。为了解决这个问题，设计出一种锁，把所有排队等锁的线程放到一个队列上，每个线程自选**自己的节点**，因为是队列，还是一个公平锁。

其算法如下：
1. 自旋锁是一个指针初始化为 `NULL`，如图中时刻 1
2. 当线程取锁时，先生成一个锁`locked=0 next=NULL`，检查自旋锁指针：
	1. 若自旋锁指针为 `NULL` ，则此前没人获得锁，将自旋锁指针指向自己的锁。如图中时刻 2
	2. 若自旋锁指针不为 `NULL`，则该锁已被获得，将此时自旋锁指向的锁的 `next` 指向自己，并更新自选锁指针也为自己的锁。然后自选变量 `locked` 直至为 1，如图中时刻 3
3. 当线程释放锁时：
	1. 若 `next` 不为 `NULL`，则将 `next->locket` 置为 `1`。如图中时刻 4。
	2. 若 `next` 为 `NULL`，则将自旋锁变量置为 `NULL`
![](https://img.jaxwang.top/2025/05/2633c01cacf10dc09abfdfb4706f9b18.png)

```
static inline
void mcs_spin_lock(struct mcs_spinlock **lock, struct mcs_spinlock *node)
{
	struct mcs_spinlock *prev;

	/* Init node */
	node->locked = 0;
	node->next   = NULL;

	prev = xchg(lock, node);     // 每次获得锁都会先将自旋锁指针指向新的锁节点
	if (likely(prev == NULL)) {  // 上述 2.1 为 NULL 直接获得锁
		return;
	}
	WRITE_ONCE(prev->next, node);

	/* Wait until the lock holder passes the lock down. */
	arch_mcs_spin_lock_contended(&node->locked);   // 上述 2.2 自旋直至 locked=1
}
```



## 三、queued spinlock

MCS 锁有一个很大的问题就是它改变了自旋锁的接口，这是一个很严重的问题，内核里使用自旋锁的地方很多，如果把自旋锁都改为MCS自旋锁，那将是非常麻烦的。同时MCS还有一个问题就是它的体积增大了，这也是一个很严重的问题。

队列自旋锁对MCS自旋锁的优化原理是，一个系统最多同时有NR_CPU个自旋锁在运行，所以没必要每个加锁线程都自己分配一个锁节点，我们在系统全局预分配NR_CPU个锁节点就可以了，哪个CPU上要执行自旋锁，就去用对应的锁节点就可以了。这是对于只有线程的情况，实际上还有软中断、硬中断、NMI，它们后者都可以抢占前者，都能抢占线程，所以整个系统实际上总共需要 NR_CPU * 4 个锁节点就足够了。

*TODO 具体算法实现后面再写了。*

## References
[1] 深入理解Linux内核之自旋锁 https://zhuanlan.zhihu.com/p/584016727

[2] spinlock与中断、抢占的关系 https://blog.csdn.net/liduxun/article/details/47833143

[3] 并发：自旋锁 https://cslqm.github.io/2020/01/06/spin-lock/