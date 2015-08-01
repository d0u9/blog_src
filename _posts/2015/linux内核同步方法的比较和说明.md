---
title: 'Linux内核同步方法的比较和说明'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
abbrlink: 3752583645
date: 2015-08-01 08:12:00
---

Linux内核给我们提供了大量的同步方法，例如spin locks，reader-writer spin locks，semaphores，reader-writer semaphores，BKL，seq locks等等。那么这些同步方法都有什么不同，在哪种情况下选择哪些方法才合适呢？

<!-- more -->

# Semaphores VS Spin Locks

**有关Spin Lock需要知道以下的几点**：

1. Spin Locks能够在中断处理函数中使用，但是这样会造成系统调度延迟。
2 .使用Spin Locks时，必须保证占用锁的时间必须要短，否则会造成系统性能的下降。如果需要长时间的占用锁，并且当前的代码执行线（不知道这样翻译对不对，英文是thread of execution）能够安全的被挂起到睡眠状态的话，可以选择使用信号量。
3. 在中断中获取Spin Locks之前必须先关闭该CPU的中断，否则有可能会造成递归的获取Spin Lock，从而造成系统死锁。
4. 正如第3条中看到的，spin lock不能递归的获取。递归的获取spin lock会造成系统的死锁。
5. 由于Spin Lock不能递归的占用，因此在所有可能发生抢占的场合，需要谨慎的处理，例如和Bottom Halves的相关操作：
   1. 由于Bottom Half会抢占进程上下文的执行。因此，如果数据在Bottom Half的进程上下文之间共享的话，则必须使用锁并且禁止Bottom Halves。
   3. 由于中断服务函数会抢占Bottom Half，因此如果数据在中断处理函数和bottom half之间共享的话，必须要获得锁并且禁止中断。
   3. 由于两个类型相同的tasklet永远不会同时运行。因此在相同类型的tasklet间共享的数据不需要保护。
   4. 如果在两个不同类型的tasklet间共享数据的话，必须在访问bottom half中数据之前获得锁。由于在同一CPU上的tasklet不会互相抢占，因此并不需要禁止bottom halves。
   5. 不管是不是相同类型的softirq，只要他们之间有数据的共享就必须被锁保护。由于即使是两个相同类型的softirq也可能在不同的CPU上同时运行。同一个CPU上的softirq并不会互相抢占，因此不需要禁止bottom halves。

**有关Semaphore需要知道以下的几点**：

1. Semaphore会导致调用它的代码执行线被挂起到睡眠队列。因此在任何不允许发生睡眠的情景下都不能使用Semaphore。
2. 如果占用Semaphore的时间很短，可以考虑使用Spin Lock，因为Semaphore会导致任务被挂起，从而增加了两次调度的开销。但是需要注意的是，如果是在单CPU机器上使用Spin Lock需要格外的小心，因为Spin的特性可能会导致系统陷入永远等待的状态。
3. Semaphore不会禁止内核的抢占，因此占用Semaphore的任务可以被其他的任务所抢占。这也就意味着，使用Semaphore不会造成系统调度的延迟。
4. 占用了Semaphore的同时不能再占用Spin Lock，因为在等待Semaphore的同时可能会睡眠，而持有Spin Lock时这是不允许的。

# Semaphores VS Mutexes

尽量使用Mutex而非Semaphore，除非针对Mutex的限制不能满足时再考虑Semaphore。

具体的限制如下：

- 只能由一个任务获取Mutex，即usage count永远是1。
- 不管是谁获取了Mutex，它必须在使用完后解锁Mutex，即不能在一个上下文中获取Mutex，而在另一个上下文中解锁Mutex。这意味着Mutex不能用在类似内核－用户空间的同步上。
- Mutex的加锁和解锁是不允许递归的，即不能递归的持有同一个锁，也不能去解锁一个已经被解锁的锁。
- 进程在获得互斥体后不能退出。
- Mutex不能在中断处理函数和Bottom Halves中使用，哪怕是mutex_trylock()函数也不行。
- Mutex只能通过官方的API来管理。

# Spin Lock VS Mutexes

| Requirement                           | Recommended Lock          |
| :------------------------------------ | :------------------------ |
| Low overhead locking                  | Spin lock is preferred.   |
| Short lock hold time                  | Spin lock is preferred.   |
| Long lock hold time                   | Mutex is preferred.       |
| Need to lock from interrupt context   | Spin lock is required.    |
| Need to sleep while holding lock      | Mutex is required.        |

# Completion Variables VS Reader-Writer Spin Locks/Semaphores

这两个的区别主要在于，Reader-Writer Spin Locks和Reader-Writer Semaphores更加的偏向于Reader，也就是说，如果Reader非常频繁的情形下，Writer可能会被处于饥饿状态。而Completion Variables会更偏向于Writer，只要有Writer时，Reader就会被无限的循环。

# 禁用抢占

加入我们的代码需要处理per-processor data，这个时候使用spin lock就显得有些多余了，因为这些数据本身就只能被单个的CPU访问，但是如果不使用spin lock，这些数据又可能因为抢占机制而被同一CPU上的其他进程所并发的访问。这个时候我们就需要禁止抢占来保护这些数据的完整性了。

使用如下的代码来禁用和启用内核抢占：

```C
preempt_disable();
/* preemption is disabled ... */
preempt_enable();
```

这个函数是可以嵌套的，但是调用了多少次`preempt_disable()`就需要调用相同次数的`preempt_enable()`。

# 顺序和屏障

有时候我们必须保证对内存操作的顺序性，尤其在多CPU系统中，对内存的读写必须按照代码启动他们的顺序而执行。由于CPU和编译器都会为了某些性能的原因对代码顺序进行调整，因此如何保证顺序是一个重要的问题。

对于如下的代码：

```C
a = 1;
b = 2;
```

无论编译器或者处理器可能都会存在打乱这两个语句顺序而执行的情形，因为在它们看来，这两个语句是不存在关系的。

`rmb()`函数提供了一个读的内存屏障。它能够保证在此之前的的读不会被乱序到它之后。
同样的`wmb()`提供了相似的写内存屏障，即任何在它之前的写不会被乱序到它之后。
`mb()`函数对读和写同时提供了内存屏障。
`rmb()`的一个变种版本函数`read_barrier_depends()`也提供了对于读的内存屏障。但是不同的是，在这个屏障之前的所有的读操作都能够被保证完成先于那些位于屏障之后的，依赖于屏障之前的读操作。换句话说，这个函数也是针对读操作的内存屏障，但是仅仅针对那些有前后依赖关系的读操作。

除了以上的函数之外，还存在着`smp_rmb()`，`smp_wmb()`，`smp_mb()`，and `smp_read_barrier_depends()`这些能够优化的宏。在SMP结构的内核中，这些宏是普通的内存屏障，在单CPU结构的机器上，这些宏被优化为编译器屏障。

`barrier()`函数能够防止编译器对跨越该函数的读操作和写操作进行优化。之前介绍的内存屏障也是编译器屏障，但是此处的`barrier()`更加的轻量和快速。

全部的屏障函数见下表：

| Barrier                      | Description                                                                |
| :--------------------------- | :------------------------------------------------------------------------- |
| rmb()                        | Prevents loads from being reordered across the barrier                     |
| read_barrier_depends()       | Prevents data-dependent loads from being re- ordered across the barrier    |
| wmb()                        | Prevents stores from being reordered across the barrier                    |
| mb()                         | Prevents load or stores from being reordered across the barrier            |
| smp_rmb()                    | Provides an rmb() on SMP, and on UP provides a barrier()                   |
| smp_read_barrier_depends()   | Provides a readbarrierdepends() on SMP, and provides a barrier() on UP     |
| smp_wmb()                    | Provides a wmb() on SMP, and provides a barrier() on UP                    |
| smp_mb()                     | Provides an mb() on SMP, and provides a barrier() on UP                    |
| barrier()                    | Prevents the compiler from optimizing stores or loads across the barrier   |

---

### ¶ The end
