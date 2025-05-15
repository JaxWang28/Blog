---
title: Linux - 内存管理 - 页描述符
slug: linux-memory-page-descriptor
share: true
draft: true
date: 2025-04-29T16:25:00+08:00
tags:
  - Linux
  - Kernel
  - Memory
categories:
---

Linux 使用分页技术，因此对物理内存管理的基本单位是 **Page Frame 页框**。Linux 必须对每一个 Page Frame 了如执掌，其通过为每个页框分配一个**页描述符** `struct page` 保存页框的基本信息。

对于一个物理地址空间 4GB PAGE_SIZE=4KB Linux 系统，Linux 分配 `4GB/4KB = 1M = 2^20` 大约一百万个 `struct page` 结构体，尽管这是个庞大的数量，但这是不可避免的。

这里我们可以得到一个对应关系:
![](https://img.jaxwang.top/2025/04/b5b82570c1935e7949d19886729d2a67.png)

Linux 中定义了两个宏进行 PFN 与 `struct page` 之间的转换：
```
page_to_pfn
pfn_to_page
```

这两个宏的具体实现依赖于 Linux 使用的物理内存模型，将在后文展开。

*include/linux/mm_types.h*
```
/*
 * Each physical page in the system has a struct page associated with
 * it to keep track of whatever it is we are using the page for at the
 * moment. Note that we have no way to track which tasks are using
 * a page, though if it is a pagecache page, rmap structures can tell us
 * who is mapping it.
 */
struct page {
    ...
    unsigned long flags;
    ...	
} _struct_page_alignment;
```


