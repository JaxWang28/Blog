---
title: Linux - 进程管理 - 进程描述符 - state
slug: linux-progress-taskstruct-state
share: true
draft: true
date: 2025-04-20T17:26:06+08:00
tags:
  - Linux
  - Kernel
  - x86
categories:
---


![](https://img.jaxwang.top/2025/04/36b9b75b2ec34273e0b4b59a2d746966.png)

```
include/linux/sched.h

struct task_struct {
    ....
	unsigned int __state;
	...
}

/* Used in tsk->__state: */
#define TASK_RUNNING            0x00000000
#define TASK_INTERRUPTIBLE      0x00000001
#define TASK_UNINTERRUPTIBLE    0x00000002
#define __TASK_STOPPED          0x00000004
#define __TASK_TRACED           0x00000008
/* Used in tsk->exit_state: */
#define EXIT_DEAD               0x00000010
#define EXIT_ZOMBIE             0x00000020
#define EXIT_TRACE              (EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->__state again: */
#define TASK_PARKED             0x00000040
#define TASK_DEAD               0x00000080
#define TASK_WAKEKILL           0x00000100
#define TASK_WAKING             0x00000200
#define TASK_NOLOAD             0x00000400
#define TASK_NEW                0x00000800
#define TASK_RTLOCK_WAIT        0x00001000
#define TASK_FREEZABLE          0x00002000
#define __TASK_FREEZABLE_UNSAFE (0x00004000 * IS_ENABLED(CONFIG_LOCKDEP))
#define TASK_FROZEN             0x00008000
#define TASK_STATE_MAX          0x00010000
#define TASK_ANY                (TASK_STATE_MAX-1)
```



|  State   | Description    |
| --- | --- |
|   TASK_RUNNING  |  可运行状态：进程要么在 CPU 上执行，要么准备执行   |
|TASK_INTERRUPTIBLE | 可中断的等待状态：进程被挂起（睡眠），直到某个条件为真。产生硬件中断，释放进程等待的系统资源，或传递一个信号都可以唤起进程。进程唤起后状态回到 TASK_RUNNING |
|TASK_UNINTERRUPTIBLE |不可中断的挂起状态：忽略任何信号。进程必须在不被打断的情况下等待某个事件的发生|
|__TASK_STOPPED | 暂停状态：进程执行被暂停，进程收到 SIGSTOP SIGTTIN SIGTSTP SIGTTOU 信号后进入暂停状态|
|__TASK_TRACED	| 跟踪状态：进程的执行已由 debugger 程序暂停。当一个进程被另一个进程监控时，每一个信号都会让进程进入此状态|
|EXIT_ZOMBIE | 僵死状态：进程的执行被终止，但是父进程还没有发布 `wait4()` 和 `waitpid()` 系统调用来返回有关死亡进程的信息。此时内核还不能丢弃死亡进程描述符中的数据，父进程还可能需要|
|EXIT_DEAD| 僵死撤销状态：父进程刚发出 `wait4()` `waitpid()` 系统调用，进程由系统删除。为了防止其它执行线程在同一个进程上也执行 `wait()` 类系统调用，把进程从 `EXIT_ZOMBIE` 切换到 `EXIT_DEAD` |

# Ref.
https://blog.jaxwang.top/p/linux-progress-taskstruct-state/