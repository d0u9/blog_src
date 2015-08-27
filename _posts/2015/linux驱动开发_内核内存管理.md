---
title: 'Linux驱动开发--内核内存管理'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 1447766914
date: 2015-08-27 15:37:00
---

# kmalloc()

关于`kmalloc()`你需要知道：

1. `kmalloc()`的实现是一个面向页的内存管理机制。这于用户空间的面向堆的`malloc()`是不同的。
2. 每次通过`kmalloc()`申请而返回的实际内存大小会往往会比请求的内存大小要大一些，但是不会超过请求内存的大小的两倍。
3. 对于`kmalloc()`能够申请的内存大小是有上限的，这与具体的硬件架构和内核配置有关，如果你想要自己的内核模块具有较好的移植性，则应该假设每次`kmalloc()`申请的内存不会超过**128kB**。
4. `kmalloc()`是建立在**lookaside cache**之上的。

<!-- more -->

# Lookaside Caches(slab layer)

1. **lookaside cache**主要是为那些频繁申请和释放相同大小数据结构的应用场景而专门设置的（例如每次创建新的进程和线程都需要创建新的`strcut task_struct`结构）。
2. **lookaside cache**的管理器被称为**slab allocator**。
3. 创建**cache**的时候，`*constructor()`和`*destructors()`是可选的。同时这两个参数的时候还有额外的限制
   - `constructor()`函数并不会在**cache**分配后立即被调用，同样`destructor()`也不会在**cache**释放后被理解调用。
   - `constructor()`和`destructor()`根据是否传入了`SLAB_CTOR_ATOMIC`标志而可以休眠也可以不休眠。
4. 使用**lookaside cache**的一个好处是内核会自动维护**cache**的使用统计。通过读取`/proc/slabinfo`可以获得这些统计数据。
5. `kmalloc()`是建立在**lookaside cache**之上的。
6. 相比使用`kmalloc()`，**lookaside cache**能够大幅的降低内存碎片。

# Memory Pools

1. **Memory Pool**是通过预先分配合适数量的对象来保留一个最低的阈值，这样当系统可用内存严重不足时，新的分配可以通过预留的空间来分配。
2. 使用**memory pool**时需要明白，我们会无故的占用一部分内存，这部分内存可能会一直不使用直到出现紧急情况。

# get_free_page和相关函数

1. **get_free_page**系列函数得到的内存在物理上是连续的，如果所请求内存的大小无法满足物理连续，则返回失败。
2. 释放的大小和申请到的大小必须相同，否则会破坏内存映射关系，从而让系统变得不稳定。
3. 对于使用这类函数，主要的优点并非是速度的提升，而是碎片的减少。使用`kmalloc()`造成的碎片会潜在的增加需对内存使用。

# vmalloc

1. `vmalloc()`分配虚拟地址连续的区域，但不一定是物理地址连续。
2. 在大多数的情形下不鼓励使用`vmlloc()`。因为通过该方法获得的内存使用效率不高，而且在某些架构上，用于`vmlloc()`的地址空间总量相对较小。
3. `kmalloc()`和`get_free_page()`获得的虚拟地址内存空间和实际的物理地址空间时一一对应的关系，仅仅只差一个偏移常量`PAGE_OFFSET`。而使用`vmalloc()`获得物理内存时，需要修改对应的**page tables**。
4. `vmalloc()`获得的地址空间是由`<asm/pgtable.h>`中的`VMALLO_START`和`VMALLOC_END`定义的。
5. 实用`vmalloc()`函数的正确场合是在分配一大块连续的、只在软件中存在的、用于缓冲的内存区域的的时候。
6. `vmalloc()`不能用在原子上下文中，因为它的内部实现调用了`kmalloc(GFP_KERNEL)`来获取页表的存储空间，因此可能会休眠。

# per-CPU变量

1. 访问per-CPU变量几乎可以不用加锁。
2. 需要谨慎处理抢占。
3. 在某些CPU架构上，per-CPU变量可使用的地址空间时受限制的。因此，如果要穿件per-CPU变量，则应该保持这些变量较小。

---

### ¶ The end
