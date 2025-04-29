---
title: Linux - 内存管理 - 物理内存模型
slug: linux-memory-physical-memory-model
share: true
draft: false
date: 2025-04-29T16:30:00+08:00
tags:
  - Linux
  - Kernel
  - Memory
categories:
---

4GB RAM 4KB 页面大小的 Linux 系统会产生一百万之多的 `struct page`，组织管理这些结构体的方式我们称为内存模型。Linux 目前支持两种模型：`FLATMEM` `SPARSEMEM`。

无论选择哪个内存模型，都会通过**一个或多个数组**管理 `struct page`。


# 内存空洞 memory hole

事实上 RAM 只是物理地址空间的一部分，物理地址空间通常会出现一部分地址是不用做普通内存使用的，对于这地址我们称之为**内存空洞 Memory Hole**。

使用 `qemu-system-x86_64 -m 8G ` 启动 Kernel，可以看到下面 log
```
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x00000000bffdffff] usable
[    0.000000] BIOS-e820: [mem 0x00000000bffe0000-0x00000000bfffffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000100000000-0x000000023fffffff] usable
[    0.000000] BIOS-e820: [mem 0x000000fd00000000-0x000000ffffffffff] reserved
```

对于其中 `reserved` 的部分是不可作为普通内存使用的。

# FLATMEM 平坦内存模型

最简单的内存模型，只适用于具有连续或大部分连续物理内存的 non-NUMA 系统。

## 基本思想

仅使用一个**全局数组** `mem_map` 来组织所有物理内存的 `struct page`。

*mm/memory.c*
```
#ifndef CONFIG_NUMA
unsigned long max_mapnr;
EXPORT_SYMBOL(max_mapnr);

struct page *mem_map;
EXPORT_SYMBOL(mem_map);
#endif
```

**Example** *物理地址空间 4MB PAGE_SIZE 4KB 的 Linux 系统*
![](https://img.jaxwang.top/2025/04/0068e2e33ea222f51b9b2c73f2db3834.png)

FLATMEM 物理内存模型，大多数架构中，内存空洞也会有对应的 entry，但是永远不会被初始化。

某些体系结构可能会释放那些不对应实际物理页面的 `mem_map` 数组部分。在这种情况下，体系结构特定的 `pfn_valid()` 实现需要考虑 `mem_map` 中的空洞。

## PFN <-> struct page 


```
PFN - ARCH_PFN_OFFSET = struct page 在 mem_map 中的索引
```
*对于一些系统其物理内存的起始地址并不是 0，PFN 与第一个页框之间存在一个偏移量 ARCH_PFN_OFFSET， 这里不做过多赘述，对于上述 Example，ARCH_PFN_OFFSET=0.*


# SPARSEMEM 稀疏内存模型

稀疏内存模型使用更灵活的管理方式，其核心思想就是对粒度更小的连续内存块进行精细的管理。

唯一支持以下特性的模型：
* hot-plug 
* hot-remove 
* alternative memory maps for non-volatile memory device
* deferred initialization of the memory map for larger systems.

## 基本思想

* 将物理内存划分为若干**段**，每个段由 `struct mem_section` 表示
* 每个段包含一个 `section_mem_map`，逻辑上指向一个 `sturct page` **数组**
* 每个段可以包含的 `struct page` 数量由 `SECTION_SIZE_BITS` 决定
* 所有的段 `struct mem_section` 被组织在一个指针数组 `mem_section` 中

```
struct mem_section {
... 
unsigned long section_mem_map; // struct page array pointer However, it is stored with some other magic.
...
}
```

```
/*
 * Permanent SPARSEMEM data:
 *
 * 1) mem_section	- memory sections, mem_map's for valid memory
 */
#ifdef CONFIG_SPARSEMEM_EXTREME
struct mem_section **mem_section;
#else
struct mem_section mem_section[NR_SECTION_ROOTS][SECTIONS_PER_ROOT]
	____cacheline_internodealigned_in_smp;
#endif
EXPORT_SYMBOL(mem_section);
```

```
include/linux/mmzone.h
#define PFN_SECTION_SHIFT       (SECTION_SIZE_BITS - PAGE_SHIFT)
#define PAGES_PER_SECTION       (1UL << PFN_SECTION_SHIFT)
```

**Example** *PAGE_SIZE 4KB 的 Linux 系统*

![](https://img.jaxwang.top/2025/04/fdda9f44385a4f7f2a896762576dc212.png)

当物理地址空间中出现 `128MB` 空洞时，并且这 `128MB` 空洞是与 128MB 对齐的就可以减少这部分的 `struct page` 分配而节省内存。

## PFN <-> struct page 

[ref](https://docs.kernel.org/mm/memory-model.html#:~:text=With%20SPARSEMEM%20there%20are%20two%20possible%20ways%20to%20convert%20a%20PFN%20to%20the%20corresponding%20struct%20page)

## INIT

```
setup_arch()
x86_init.paging.pagetable_init()
native_pagetable_init()
paging_init()
sparse_init()
```


*mm/sparse.c*
```
/*
 * Allocate the accumulated non-linear sections, allocate a mem_map
 * for each and record the physical to section mapping.
 */
void __init sparse_init(void)
{
	unsigned long pnum_end, pnum_begin, map_count = 1;
	int nid_begin;

	/* see include/linux/mmzone.h 'struct mem_section' definition */
	BUILD_BUG_ON(!is_power_of_2(sizeof(struct mem_section)));
	memblocks_present(); // 根据 memory blocks malloc struct mem_section, 暂不 malloc struct page

	pnum_begin = first_present_section_nr();
	nid_begin = sparse_early_nid(__nr_to_section(pnum_begin));

	/* Setup pageblock_order for HUGETLB_PAGE_SIZE_VARIABLE */
	set_pageblock_order();

	for_each_present_section_nr(pnum_begin + 1, pnum_end) {
		int nid = sparse_early_nid(__nr_to_section(pnum_end));

		if (nid == nid_begin) {
			map_count++;
			continue;
		}
		/* Init node with sections in range [pnum_begin, pnum_end) */
		sparse_init_nid(nid_begin, pnum_begin, pnum_end, map_count);  // 遍历所有 section 并根据 node id 分组 init, malloc struct page.
		nid_begin = nid;
		pnum_begin = pnum_end;
		map_count = 1;
	}
	/* cover the last node */
	sparse_init_nid(nid_begin, pnum_begin, pnum_end, map_count);
	vmemmap_populate_print_last();
}

```



# Ref
https://docs.kernel.org/mm/memory-model.html
https://www.cnblogs.com/binlovetech/p/16914715.html