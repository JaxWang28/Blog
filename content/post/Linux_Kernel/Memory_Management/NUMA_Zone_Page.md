---
title: Linux - 内存管理 - NUMA ZONE
slug: linux-memory-numa-zone-page
share: true
draft: true
date: 2025-05-07T20:25:00+08:00
tags:
  - Linux
  - Kernel
  - Memory
categories:
---

# 0x01 NUMA Non-Uniform Memory Access 概述
![](https://img.jaxwang.top/2025/04/54d2c03b18a4e0eab8c71c100f7a820f.png)

## 单处理器时代
单核时代，CPU 与内存之间的关系是简单的。CPU 通过前端总线（FSB, Front Side Bus）连接到北桥芯片（集成内存控制器），北桥芯片连接到内存。
![](https://img.jaxwang.top/2025/04/f49b62a3a29bc162d11b5bf6e1851693.png)
## 多处理器时代

随着 CPU 的频率提高出现瓶颈，开始向多核发展。 出现了对称多处理器系统，多个 CPU 共享同一个内存池，通过总线 Bus 相连。每个 CPU 访问内存所需时间相同。这种架构称为 **Uniform-memory-Access UMA**。使用总线就会存在资源争用和一致性问题，随着 CPU 数量增多争用愈演愈烈，以至于 4 核 CPU 性能达不到 2 核 1.5 倍。


放弃总线的访问方式，将 CPU 划分到多个 Node 中，每个 Node 有自己独立的内存空间。各个 Node 之间通过高速互联通讯，通讯通道被成为QuickPath Interconnect 即 QPI。


这种架构模型，cpu访问本地内存与远程内存所用的时间是不一样的，一般访问本地内存要比访问远程内存快，因此也被称做非一致或非均匀内存访问模型(Nonuniform-Memory-Access, 简称NUMA)。在 NUMA 中，每个 CPU（或 CPU 组）有自己的本地内存，同时也可以访问其他 CPU 的远程内存，但远程访问的延迟更高。

![](https://img.jaxwang.top/2025/04/3da9e0613ee79b63f12ab97135bac08d.png)

*一颗 CPU 一定在一个 Node 中。一个 Node 可以包含多颗 CPU。*

# 0x02 pg_data_t

Linux 在 2.6.7 版本引入 NUMA，并作为 Linux 物理内存管理的第一原则。这就意味着即使系统架构为 UMA 架构，Linux 也会将其视为 NUMA，只不过是只有一个 Node 的 NUMA 架构。

Linux 使用 `struct pglist_data` 描述一个 Node 上的内存。可以通过 `NODE_DATA(nid)` 宏找到指定 `nid` 对应的 `pglist_data`。

*include/linux/mmzone.h*
```
/*
 * On NUMA machines, each NUMA node would have a pg_data_t to describe
 * it's memory layout. On UMA machines there is a single pglist_data which
 * describes the whole memory.
 *
 * Memory statistics and page replacement data structures are maintained on a
 * per-zone basis.
 */
typedef struct pglist_data {
    ....
} pg_data_t;
```

*这里并不是真正管理物理内存的 struct page, stuct page 由物理内存模型管理，pglist_data 巧妙的使用了 list 这个词，意在只是引用。*

对于 UMA 架构，Linux 只会创建一个 `struct pglist_data`，其变量名为 `contig_page_data`。

*mm/memblock.c*
```
#ifndef CONFIG_NUMA
struct pglist_data __refdata contig_page_data;
EXPORT_SYMBOL(contig_page_data);
#endif
```

在分配物理页时，Linux 默认采用“节点本地”策略：优先从距离当前 CPU 最近的节点上分配内存。因为进程往往长期运行在同一颗 CPU 上，所以通常会使用该 CPU 所在节点的内存。若需要，也可以按“NUMA 内存策略”文档所述，由用户显式控制分配策略。



# 0x03 zone

在**理想**的计算机系统中，一个页框就是一个基本存储单元，可以用来存放数据代码，任意一个页框的使用都没有限制。但事实并非如此。受限于计算机体系结构的硬件制约和某些新特性（*TODO*），页框的使用方式也得到了限制。

*include/linux/mmzone.h*
```
struct zone {
	...
	struct pglist_data	*zone_pgdat;
	...
}
```

*include/linux/mmzone.h*
```
enum zone_type {
#ifdef CONFIG_ZONE_DMA
	ZONE_DMA,
#endif
#ifdef CONFIG_ZONE_DMA32
	ZONE_DMA32,
#endif
	ZONE_NORMAL,
#ifdef CONFIG_HIGHMEM
	ZONE_HIGHMEM,
#endif
	ZONE_MOVABLE,
#ifdef CONFIG_ZONE_DEVICE
	ZONE_DEVICE,
#endif
	__MAX_NR_ZONES
};
```

**ZONE 分类**：
- **ZONE_DMA** 和 **ZONE_DMA32**：  
    这两个 zone 在历史上用于表示可供**外设设备**进行 DMA 访问的内存，  
    这些设备无法访问整个物理内存地址空间。  
    虽然多年以后出现了更好、更健壮的接口（如基于通用设备的动态 DMA 映射），  
    但 **ZONE_DMA** 和 **ZONE_DMA32** 仍然用于表示那些访问受限的内存范围。  
    根据不同架构，**可以在编译时**（build time）通过配置选项 **CONFIG_ZONE_DMA** 和 **CONFIG_ZONE_DMA32** 禁用其中之一或两者。  
    某些 64 位平台可能需要同时启用这两个 zone，因为它们支持不同 DMA 地址限制的外设。
    
- **ZONE_NORMAL**：  
    表示常规内存，内核可以随时访问。  
    如果 DMA 设备可以访问全部地址空间，DMA 操作也可以在此 zone 的页面上进行。  
    **ZONE_NORMAL 总是启用的**。
    
- **ZONE_HIGHMEM**：  
    是物理内存中未在内核页表中永久映射的部分。  
    内核只能通过**临时映射（temporary mapping）**来访问这部分内存。  
    该 zone 仅出现在一些 32 位架构上，并且通过 **CONFIG_HIGHMEM** 配置项启用。
    
- **ZONE_MOVABLE**：  
    也是常规可访问的内存，就像 **ZONE_NORMAL**。  
    不同之处在于，**ZONE_MOVABLE** 中大部分页面的内容是**可移动的**。  
    即，虽然这些页面的虚拟地址保持不变，但它们对应的物理页面内容可以迁移。  
    通常 **ZONE_MOVABLE** 在内存热插拔（memory hotplug）时被填充，  
    也可以在系统启动时通过内核命令行参数（如 `kernelcore`、`movablecore`、`movable_node`）进行配置。  
    详情可参考页面迁移（Page migration）和内存热插拔（Memory Hot(Un)Plug）章节。
    
- **ZONE_DEVICE**：  
    表示驻留在设备（如 PMEM 持久内存设备或 GPU 显存）上的内存。  
    它与传统 RAM 区域类型有不同的特性，存在的目的是为设备驱动标识的物理地址范围提供 `struct page` 和内存映射（memory map）服务。  
    **ZONE_DEVICE** 通过配置选项 **CONFIG_ZONE_DEVICE** 启用。
    
特别需要注意的是，很多内核操作只能在 **ZONE_NORMAL** 中进行，因此 **ZONE_NORMAL 是性能最关键的内存区域**。



# 0x04 Node Zone Page 组织结构

![](https://img.jaxwang.top/2025/05/ec28f3a25fc0fc066700823373ab4d8c.png)

*include/linux/mmzone.h*
```
typedef struct pglist_data {
	...
	/*
	 * node_zones contains just the zones for THIS node. Not all of the
	 * zones may be populated, but it is the full list. It is referenced by
	 * this node's node_zonelists as well as other node's node_zonelists.
	 */
	struct zone node_zones[MAX_NR_ZONES];
	...
}

struct zone {
	...
	struct per_cpu_pages	__percpu *per_cpu_pageset;

	/* zone_start_pfn == zone_start_paddr >> PAGE_SHIFT */
	unsigned long		zone_start_pfn;
	unsigned long		spanned_pages;
	
	/* free areas of different sizes */
	struct free_area	free_area[NR_PAGE_ORDERS];
	...
}

struct free_area {
	struct list_head	free_list[MIGRATE_TYPES];
	unsigned long		nr_free;
};

struct per_cpu_pages {

	short free_count;	/* consecutive free count */

	/* Lists of pages, one per migrate type stored on the pcp-lists */
	struct list_head lists[NR_PCP_LISTS];
} ____cacheline_aligned_in_smp;
```

zone 里的空闲页要么位于 `Per-CPU Pageset(PCP)` 要么位于 该 zone 的 `全局空闲链表 free areas`。


