---
title: Linux Kernel 内核同步之自旋锁
description: Linux Kernel 同步技术之自旋锁原理分析。
slug: linux-kernel-synchronization-spinlock
share: true
draft: false
date: 2025-05-19T22:37:06+08:00
tags:
  - Kernel
  - KernelSynchronization
categories: Kernel
---

# Linux Kernel 内核同步之自旋锁

**原子操作只能对整形变量或者位进行保护**，事实上，临界区不是只有整型变量和位这么简单。加锁（locking）是另一种广泛应用的同步技术。当要访问**共享数据结构**或进入**临界区**时，为自己获得一把“锁”，离开时再释放锁。在获得锁到释放锁之间的时间内是禁止其他进程访问的。

```
spinlock_t lock;     // 定义自旋锁

spin_lock(&lock);    // 上锁
/* 临界区 */
spin_unlock(&lock);  // 释放锁
```

## 一、Linux Kernel 自旋锁 API

| API                                    | Description                                                  |
| -------------------------------------- | ------------------------------------------------------------ |
| `DEFINE_SPINLOCK(spinlock_t lock)`     | 定义并初始化一个自选变量。                                   |
| `int spin_lock_init(spinlock_t *lock)` | 初始化自旋锁。                                               |
| `void spin_lock(spinlock_t *lock)`     | 获取指定的自旋锁，也叫做加锁。                               |
| `void spin_unlock(spinlock_t *lock)`   | 释放指定的自旋锁。                                           |
| `int spin_trylock(spinlock_t *lock)`   | 尝试获取指定的自旋锁，如果没有获取到就返回 0                 |
| `int spin_is_locked(spinlock_t *lock)` | 检查指定的自旋锁是否被获取，如果没有被获取就<br>返回非 0，否则返回 0。 |

上述 API 自动禁止内核抢占，但是不会禁用本地中断。**当需要上锁的共享资源会在中断处理中被访问，我们需要使用禁止本地中断的锁**。否则，当线程 A 上锁内核资源后触发中断，中断程序中获得相同的锁，会造成死锁。为此 Kernel 提供了以下 API：

| API                                                          | Description                                             |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| `void spin_lock_irq(spinlock_t *lock)`                       | 禁止本地中断，并获取自旋锁。                            |
| `void spin_unlock_irq(spinlock_t *lock)`                     | 激活本地中断，并释放自旋锁。                            |
| `void spin_lock_irqsave(spinlock_t *lock,unsigned long flags)` | 保存中断状态，禁止本地中断，并获取自旋锁。              |
| `void spin_unlock_irqrestore(spinlock_t *lock, unsigned long flags)` | 将中断状态恢复到以前的状态,并且激活本地中断，释放自旋锁 |

`spin_lock_irq()` **在任何情况下都是安全的**，但是没有 `spin_lock()` 快。[3]

建议使用 `spin_lock_irqsave` / `spin_unlock_irqrestore`，因为这一组函数会保存中断状态，在释放锁的时候会恢复中断状态。一般在线程中使用 `spin_lock_irqsave` / `spin_unlock_irqrestore`

## 二、自旋锁注意事项

1. **自旋**：等待自旋锁的线程会一直处于自旋状态，这样会浪费处理器时间，降低系统性能，所以自旋锁的持有时间不能太长。自旋锁适用于短时期的轻量级加锁，如果遇到需要长时间持有锁的场景那就需要换其他的方法了
2. **睡眠**：被自旋锁保护的临界区一定不能调用任何能够引起睡眠和阻塞的API 函数，否则的话会可能会导致死锁现象的发生。如果线程 A 在持有锁期间进入了休眠状态，那么线程 A 会自动放弃 CPU 使用权。线程 B 开始运行，线程 B 也想要获取锁，但是此时锁被 A 线程持有。此时 线程 B 只能空等，直到下一次调度到线程 A 并释放资源。注意：**线程 B 在获得锁的自旋期间是可抢占的**。这有效的避免了死锁。
3. **中断**：中断里面可以使用自旋锁，但是在中断里面使用自旋锁的时候，在获取锁之前一定要先禁止本地中断(也就是本 CPU 中断，对于多核 SOC来说会有多个 CPU 核)，否则可能导致锁死现象的发生
4. **下半部**：下半部(BH)也会竞争共享资源，如果要在下半部里面使用自旋锁，可以使用下面 API：

| API                                         | Description                |
| ------------------------------------------- | -------------------------- |
| `void spin_lock_bh(spinlock_t *lock)`       | 关闭下半部，并获取自旋锁。 |
| `void spin_unlock_bh(spinlock_t *lock)`<br> | 打开下半部，并释放自旋锁。 |

## 三、自旋锁衍生

| 自旋锁   | 读写锁         | 顺序锁                     |
| -------- | -------------- | -------------------------- |
| 绝对互斥 | 读共享，写独占 | 写者优先，读者乐观验证<br> |

### 读写锁

**读锁 (Read Lock)**: 多个线程可以同时持有读锁。一个线程在请求读锁时，如果当前没有写者持有锁，它就可以立即获得锁并进入临界区。

**写锁 (Write Lock)**: 写锁是完全互斥的。一个线程在请求写锁时，必须等待所有读者和任何已存在的写者都释放锁之后，才能获得锁。当一个写者持有锁时，其他任何线程（无论是读者还是写者）都必须等待。

核心思想：**读共享，写独占 (Shared-Read, Exclusive-Write)**

### 顺序锁

顺序锁是一种为“读多写少”场景设计的、给予写者极高优先级的轻量级同步机制。它允许读者和写者并发执行，读者通过一个序列计数器来检测在读取期间是否有写者修改了数据。

顺序锁的设计哲学非常激进，它建立在几个关键假设之上：

1. 写者至上 (Writers are rare and fast): 写入操作非常少，但一旦发生，必须以最快的速度完成，不能被任何读者拖延。
2. 读者乐观 (Readers are optimistic): 读者乐观地认为，在它读取数据的短暂瞬间，不太可能会有写者闯入。因此，它不加锁就直接去读。
3. 事后验证 (Verification after the fact): 读者在读完后，必须通过一种机制来验证自己的“乐观”是否正确。这个机制就是序列计数器。
   - 写者进入前，序列号+1（变为奇数）。
   - 写者退出后，序列号再+1（变回偶数）。
   - 读者读取前后检查序列号，如果序列号未变且为偶数，则读取有效。否则，读取的数据可能被“污染”，必须重试。

核心理念：**写者优先，读者乐观验证 (Writer-Priority, Optimistic-Read-with-Verify)**

## 三、自旋锁的发展

| 代数   | 名称            | 内核版本      | 公平性 | 高效性 | 接口兼容性 |
| ------ | --------------- | ------------- | ------ | ------ | ---------- |
| 第一代 | 原始自旋锁      | 1.0 - 2.6.24  | 不公平 | 不高效 | 兼容       |
| 第二代 | ticket spinlock | 2.6.25 - 4.1x | 公平   | 不高效 | 兼容       |
| 第三代 | MCS spinlock    | 3.15 - now    | 公平   | 高效   | 不兼容     |
| 第四代 | queued spinlock | 4.2 - now     | 公平   | 高效   | 兼容       |

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



### MCS spinlock

ticket spinlock 完美解决了公平问题，但是存在一个问题，就是当锁竞争比较激烈的时候，大家都在自旋同一个变量，造成 **多核频繁竞争同一个缓存行**，引发 **缓存颠簸**，进而拖慢整个系统的同步效率。为了解决这个问题，设计出一种锁，把所有排队等锁的线程放到一个队列上，每个线程自选**自己的节点**，因为是队列，还是一个公平锁。

其算法如下：

1. 自旋锁是一个指针初始化为 `NULL`，如图中时刻 1

2. 当线程取锁时，先生成一个锁

   ```
   locked=0 next=NULL
   ```

   ，检查自旋锁指针：

   1. 若自旋锁指针为 `NULL` ，则此前没人获得锁，将自旋锁指针指向自己的锁。如图中时刻 2
   2. 若自旋锁指针不为 `NULL`，则该锁已被获得，将此时自旋锁指向的锁的 `next` 指向自己，并更新自选锁指针也为自己的锁。然后自选变量 `locked` 直至为 1，如图中时刻 3

3. 当线程释放锁时：

   1. 若 `next` 不为 `NULL`，则将 `next->locket` 置为 `1`。如图中时刻 4。
   2. 若 `next` 为 `NULL`，则将自旋锁变量置为 `NULL`

![img](https://img.jaxwang28.top/2025/05/2633c01cacf10dc09abfdfb4706f9b18.png)

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

### queued spinlock

MCS 锁有一个很大的问题就是它改变了自旋锁的接口，这是一个很严重的问题，内核里使用自旋锁的地方很多，如果把自旋锁都改为MCS自旋锁，那将是非常麻烦的。同时MCS还有一个问题就是它的体积增大了，这也是一个很严重的问题。

队列自旋锁对MCS自旋锁的优化原理是，一个系统最多同时有NR_CPU个自旋锁在运行，所以没必要每个加锁线程都自己分配一个锁节点，我们在系统全局预分配NR_CPU个锁节点就可以了，哪个CPU上要执行自旋锁，就去用对应的锁节点就可以了。这是对于只有线程的情况，实际上还有软中断、硬中断、NMI，它们后者都可以抢占前者，都能抢占线程，所以整个系统实际上总共需要 NR_CPU * 4 个锁节点就足够了。

*TODO 具体算法实现后面再写了。*



## References

[1] 深入理解Linux内核之自旋锁 https://zhuanlan.zhihu.com/p/584016727

[2] spinlock与中断、抢占的关系 https://blog.csdn.net/liduxun/article/details/47833143

[3] 并发：自旋锁 https://cslqm.github.io/2020/01/06/spin-lock/