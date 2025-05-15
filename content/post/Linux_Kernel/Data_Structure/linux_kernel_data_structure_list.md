---
title: Linux Kernel 数据结构分析之链表
slug: linux-data-structure-list
share: true
draft: false
date: 2025-04-30T15:07:00+08:00
tags:
  - Kernel
  - DataStructure
categories:
---


# Linux Kernel 数据结构分析之链表

Linux Kernel 中实现了以下两种链表：
* 双向循环链表
* hash 链表

本文将分别分析两种链表的实现。

## 一、双向循环链表

Linux 使用了最简洁的方式实现了一个几乎是万能的链表，其通过将 `struct list_head` **嵌入到其他结构体中**，实现双向循环链表。并在 `include/linux/list.h` 定义了支持的所有操作。

```C
// include/linux/types.h

struct list_head {
	struct list_head *next, *prev;
};
```


以 `keme_caches` 链表进行分析。

```
// mm/slab_common.c

LIST_HEAD(slab_caches);
```

```
// mm/slab.h

struct kmem_cache {
    ....
    struct list_head list;		/* List of slab caches */
    ....
}
```

<center><img src="https://img.jaxwang.top/2025/04/c8590fa84ab49e876bb5296f2dc13712.png" width="70%" height="70%"> </center>

上面的例子中，指针指向是 `struct kmem_cache` 中 `list` 成员。通过这个成员的地址获得 `struct kmem_cache` 的地址的操作，在 Linux 中通过 `list_entry` 实现。下面分析其实现逻辑。

```
// include/linux/list.h

/**
 * list_entry - get the struct for this entry
 * @ptr:	the &struct list_head pointer.
 * @type:	the type of the struct this is embedded in.
 * @member:	the name of the list_head within the struct.
 */
#define list_entry(ptr, type, member) \
	container_of(ptr, type, member)
```

```
// include/linux/container_of.h

/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:	the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:	the name of the member within the struct.
 *
 * WARNING: any const qualifier of @ptr is lost.
 */
#define container_of(ptr, type, member) ({				\
	void *__mptr = (void *)(ptr);					\
	static_assert(__same_type(*(ptr), ((type *)0)->member) ||	\
		      __same_type(*(ptr), void),			\
		      "pointer type mismatch in container_of()");	\
	((type *)(__mptr - offsetof(type, member))); })
```

```
// tools/include/linux/kernel.h

#ifndef offsetof
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#endif
```

1. `offsetof` 计算结构体中某个成员的偏移量，其通过强转假设地址 0 处有一个 TYPE 类型的结构体，`->`指针取结构体成员 Member，最后通过 `&` 获得 Member 的地址，并强转为 `size_t` 类型。
2. `container_of` 利用成员指针，获得结构体的指针。**成员指针 - 成员的偏移量=结构体指针**
3. `list_entry` list_head 作为一个成员嵌入到结构体中，已知 list_head 指针，调用 `container_of` 获得结构体指针。


## 二、hash 链表

linux kernel 中定义 `hlist_head` 用作 hash 表中的链表头，`hlist_node` 用作链表中的某一项。

```
// include/linux/types.h

struct hlist_head {
	struct hlist_node *first;
};

struct hlist_node {
	struct hlist_node *next, **pprev;
};
```

<center><img src="https://img.jaxwang.top/2025/04/743a01bbad60899615e2f1507da5f383.png" width="70%" height="70%"></center>

Linux kernel 设计 `hlist_head` 仅包含一个指针用作 hash_table 中的列表头，这样可以节省很大空间，特别是当 hash bucket 很大的时候，可以节省一半空间。

散列表的冲突情况很少，链表不会很长，遍历很快。将 `pprev` 设计为指向前一个节点的 `next` 很容易实现删除操作。

```
// include/linux/list.h

static inline void __hlist_del(struct hlist_node *n)
{
	struct hlist_node *next = n->next;
	struct hlist_node **pprev = n->pprev;

	WRITE_ONCE(*pprev, next);
	if (next)
		WRITE_ONCE(next->pprev, pprev);
}
```


# Ref
https://blog.csdn.net/weixin_39094034/article/details/104803967

https://linux.laoqinren.net/kernel/hlist/
