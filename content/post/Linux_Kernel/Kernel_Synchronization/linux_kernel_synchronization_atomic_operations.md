---
title: Linux Kernel 内核同步之原子操作
slug: linux-kernel-synchronization-atomic-operations
share: true
draft: false
date: 2025-05-18T21:37:06+08:00
tags:
  - Kernel
  - KernelSynchronization
categories:
---

# Linux Kernel 内核同步之原子操作

原子操作是指指令以原子的方式执行，执行过程不会被打断。要保证操作的原子性和完整性，需要“原子地”（不间断地）完整**读-修改-回写**机制。

```
static int i = 0;
void thread_A _func()
{
	i++;
}
void thread_B _func()
{
	i++;
}
```

上面代码的理想执行结果是`i` 为 2，但事实并非如此， i 在有一定的概率为 1。对于 `thread_A` 和 `thread_B` 进行的操作可以简单理解为：**读 - 修改 - 回写**。从单处理器和多处理器角度分析：
* 单处理器：`thread_A` 与 `thread_B` 存在并发，当 `thread_A` 执行完 `读` 操作，CPU 有可能切换为 `thread_B` 执行 `读` 操作，这样最终的结果为 1.
* 多处理器：`thread_A` 与 `thread_B` 存在并发，`thread_A` 与 `thread_B` 可能同时执行 `读` 操作，这样最终结果也为 1.

因此，要想保证结果的正确性，我们必须保证上面的操作完整地原子地（不间断地）完成**读-修改-回写**。

原子操作是以硬件为基础，即 **CPU 必须提供原子操作的汇编指令**。本文以 ARMv8 架构为例从下至上分析原子操作。

## ARM64 原子操作

原子操作需要处理器提供硬件支持，ARMv8 提供两种方式实现了原子操作：
1. 独占内存访问指令：独占加载（Load-Exclusive）和独占存储（Store-Exclusive）指令，其实现方式叫做连接加载/条件存储（Load-Link/Store-Conditional, LL/SC）
2. 原子内存访问指令：ARMv8.1 体系实现的 LSE 指令

*本文不对这两种方式详细展开，感兴趣的可以阅读 References[2] 书籍。*

### 独占内存访问指令 LD/SC

`LL/LC` 机制可分为两部分：
1. `LL` 从指定内存地址读取一个值，处理器会监控这个内存地址，监视其它处理器是否会修改这个地址
2. `SC` 尝试写入，若这段时间内其他处理器没有修改该内存地址，则写入成功。否则 SC 失败则重新开始整个过程

注意：`LL` 与 `SC` 之间是原子的，因此执行 `LL/SC` 指令的处理器不会进行任务调度，因此这里仅需要监视其它处理器。

`ll/LC` 在 ARMv8 指令集中的具体实现为：`LDXR` `STXR`：
* `LDXR`：独占内存加载指令，其以独占方式加载内存地址的值到通用寄存器
* `STXR`：独占内存存储指令，其以独占方式把数据存储到内存中

```ASM
ldxr <xt>， [xn | sp]
```
以上是 `LDXR` 原型，其把 `Xn` 或 `SP` 地址的值原子地加载到 `Xt` 寄存器里。

```ASM
stxr <ws> <xt> [xn | sp]
```
以上是 `STXR` 原型，其**把 `Xt` 寄存器的值原子地存储到 `Xn` 或者 `SP` 地址里，执行的结果反映到 `Ws` 寄存器中**：
* 若 `Ws` 寄存器的值为 0, 则说明这段时间内没有其它处理器修改过该内存地址中的内容，写入成功。
* 若 `Ws` 寄存器的值为 1, 则说明这段时间内有其它处理器修改过该内存地址中的内容，写入失败，此时需要跳转到 `LDXR` 处重复上述过程。

以下是一个使用独占内存访问指令实现的原子加法：
```
typedef struct {
	int counter;
} atomic_t;
 // LXDR 独占方式加载
void atomic_add(int i, atomic_t *v)
{
	unsigned long tmp;
	int result;
	asm volatile("// atomic_add\n"  
	"1: ldxr%w0, [%2]\n"            // 读：原子加载 v->counter
	" add%w0, %w0, %w3\n"           // 修改：+ i
	" stxr%w1, %w0, [%2]\n"         // 回写：原子存储到 v->counter
	" cbnz%w1, 1b"                  // 如果失败则重复上面过程
	: "=&r" (result), "=&r" (tmp)
	: "r" (&v->counter), "Ir" (i)
	: "cc");
}
```

### 原子内存访问操作指令

在 ARMv8.1 体系结构中新增了原子内存访问操作指令 atomic memory access instruction，也被称为 LSE Large System Extension

本文不对细节进行深究，只需知道下面指令

原子加载（atomic load）指令：加载地址中的值并保存，做运算

原子存储（atomic store）指令，先运算，然后原子地存储。
上述两类指令的执行过程都是原子性的


LSE 提供三类指令：
* 比较并交换 Compare And Swap CAS 指令
* 原子内存访问指令：
	1. 原子加载指令：加载地址中旧值，做运算，回写新值，**同时返回旧值**。 *注意这里的加载并不是值仅加载一个值。而是指会返回旧值。*
	2. 原子存储指令：加载地址中旧值，做运算，回写新值。
* 交换指令


```
ld<op> <xs>, <xt>, [<xn|sp>]
```
以上为原子加载指令的原型，ld 后紧跟操作，其表示：
```
tmp = *xn        // 取 xn 地址中的值
*xn = *xn op xs  // 与 xs 寄存器中的值 op 操作后回写
xt = tmp         // 返回旧值
```

|原子操作后缀|说 明|
| --- | --- |
|   add  |  加法运算   |
|clr|清零|
|set|置位|
|eor|异或操作|
|smax|有符号数的最大值操作|
|smix|有符号数的最小值操作|
|umax|无符号数的最大值操作|
|umix|无符号数的最小值操作|


## Linux Kernel 原子操作

接下来从 Linux 源码角度分析原子操作。正如前文所言 ARMv8 支持两种形式的原子操作 `LD/SC` 和 `LSE`，在 Linux Kernel 中的体现如下：

```
// arch/arm64/include/asm/atomic.h

#define ATOMIC_OP(op)							                \
static __always_inline void arch_##op(int i, atomic_t *v)		\
{									                            \
	__lse_ll_sc_body(op, i, v);					                \
}
```

```
// arch/arm64/include/asm/lse.h

#define __lse_ll_sc_body(op, ...)					        \
({									                        \
	alternative_has_cap_likely(ARM64_HAS_LSE_ATOMICS) ?		\
		__lse_##op(__VA_ARGS__) :				            \
		__ll_sc_##op(__VA_ARGS__);				            \
})
```

对于一个原子操作 `op` 内核会在 `__lse_ll_sc_body` 中判断其是否支持 `LSE`，如果支持则使用 `LSE` 方式，接下来我们分别从 `LSE` 和 `LD/SC` 角度分析。

**Linux Kernel 原子操作的 LD/SC 实现**

```
// arch/arm64/include/asm/atomic_ll_sc.h


/*
 * AArch64 UP and SMP safe atomic ops.  We use load exclusive and
 * store exclusive to ensure that these are atomic.  We may loop
 * to ensure that the update happens.
 */

#define ATOMIC_OP(op, asm_op, constraint)				\
static __always_inline void						        \
__ll_sc_atomic_##op(int i, atomic_t *v)					\
{									                    \
	unsigned long tmp;						            \
	int result;							                \
									                    \
	asm volatile("// atomic_" #op "\n"				    \
	"	prfm	pstl1strm, %2\n"				        \
	"1:	ldxr	%w0, %2\n"					            \
	"	" #asm_op "	%w0, %w0, %w3\n"			        \
	"	stxr	%w1, %w0, %2\n"					        \
	"	cbnz	%w1, 1b\n"					            \
	: "=&r" (result), "=&r" (tmp), "+Q" (v->counter)	\
	: __stringify(constraint) "r" (i));				    \
}
```


**Linux Kernel 原子操作的 LSE 实现**

```
// arch/arm64/include/asm/lse.h

#define __LSE_PREAMBLE	".arch_extension lse\n"
```

```
// arch/arm64/include/asm/atomic_lse.h

#define ATOMIC64_OP(op, asm_op)						    \
static __always_inline void						        \
__lse_atomic64_##op(s64 i, atomic64_t *v)				\
{									                    \
	asm volatile(							            \
	__LSE_PREAMBLE							            \
	"	" #asm_op "	%[i], %[v]\n"				        \
	: [v] "+Q" (v->counter)						        \
	: [i] "r" (i));							            \
}
```


## References

[1] Bovet, Daniel P., and Marco Cesati. _Understanding the Linux Kernel_. 3rd ed, O’Reilly, 2006.

[2] 奔跑吧Linux社区.ARM64体系结构编程与实践.2022.