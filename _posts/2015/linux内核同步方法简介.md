---
title: '内核同步方法简介'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
abbrlink: 3699999470
date: 2015-08-01 05:34:00
---

Linux驱动和内核的开发相比较用户空间的代码，最显著的就是对于各类并发的控制。在用户空间中。

与普通用户空间下的进程间同步不同的是，由于整个内核是一个整体，一个业余的驱动也可能导致整个内核的锁死。除此之外，内核中充满了大量的异步特性，例如多处理器，软硬件中断，内核抢占都可能导致资源产生竞争。因此掌握好内核的同步机制，是学习驱动和内核开发的先决条件。

<!-- more -->

---

# 目录
- [Atomic Operations](#Atomic-Operations)
    - [Atomic Integer Operations](#Atomic-Integer-Operations)
    - [64bit Atomic Operations](#64bit-Atomic-Operations)
    - [Atomic Bitwise Operations](#Atomic-Bitwise-Operations)
- [Spin Locks](#Spin-Locks)
    - [Spin Lock Methods](#Spin-Lock-Methods)
    - [与Spin Lock有关的函数](#与Spin-Lock有关的函数)
    - [Spin Lock & Bottom Halves](#Spin-Lock-amp-Bottom-Halves)
    - [Reader-Writer Spin Locks](#Reader-Writer-Spin-Locks)
    - [Spin Lock总结](#Spin-Lock总结)
- [Semaphores(信号量)](#Semaphores-信号量)
    - [Counting and Binary Semaphores](#Counting-and-Binary-Semaphores)
    - [Creating and Initializing Semaphore](#Creating-and-Initializing-Semaphore)
    - [信号量的使用](#信号量的使用)
    - [Reader-Writer Semaphores](#Reader-Writer-Semaphores)
- [Mutex](#Mutex)
- [Completion Variables](#Completion-Variables)
- [BKL: The Big Kernel Lock](#BKL-The-Big-Kernel-Lock)
- [Sequential Locks](#Sequential-Locks)

---

# Atomic Operations

## Atomic Integer Operations

Atomic整数操作主要使用的是一个叫做`atomic_t`的特殊类型。`atomic_t`类型定义在`<linux/types.h>`中:

```C
typedef struct {
    volatile int counter;
} atomic_t;
```

虽然`atomic_t`类型的实质是`int`，既在所有的CPU架构中都位32位，但内核开发者却假定`atomic_t`类型最大不超过24位。实际上，造成这种现象的原因是由于SPARC架构的CPU无法提供合适机器指令级别的并发访问保护。因此，在`atomic_t`类型的低8位中嵌入了一个锁,用于保护对该类型变量的并发保护。虽然在其他的架构上可以使用全部的32位，但是为了移植的要求，我们依然保持了仅仅只用其中的24位的习惯。不过最近聪明的内核黑客已经允许在SPARC架构的CPU上使用全部的32位`atomic_t`类型，因此**上述限制已经不存在了**。

对于`atomic_t`变量的操作都在`<asm/atomic.h>`中声明了。部分的CPU架构提供了额外的方法用以操作`atomic_t`变量，但是所有的架构都会提供一个最小的函数集用以维护`atomic_t`类型的变量。

`atomic_t`变量的一个主要作用是作为计数器，从而避免了使用复杂锁机制。

该类型的一个其他应用场景是原子的进行一个操作并测试结果，例如自减后查看是否为`0`。

```C
int atomic_dec_and_test(atomic_t *v)
```

上面的函数会对给定的`atomic_t`类型进行减1操作，同时判断结果是否为`0`。如果为`0`则返回`true`，否则返回`false`。

针对`atomic_t`的所有操作的完整列表可参考`<asm/atomic.h>`。

在我们的代码中，尽量选择使用`atomic_t`类型，而非其他的复杂的锁机制。在大多数的架构上，对于一两个`atomic_t`类型变量的操作不会引起过多的开销和cache-line thrashing。然而，针对性能敏感的代码，测试不同的方法是最明智的。

## 64bit Atomic Operations

随着64位架构和系统的普及，内核开发者也提出了针对64位版本的atomic_t变量的变种版本`atomic64_t`。几乎所有的针对32位`atomic_t`类型的操作函数都有其对应的64位版本，只是他们用`atomic64`前缀替换了`atomic`前缀。

## Atomic Bitwise Operations

除了对整个`atomic_t`类型的操作之外，内核还提供了一套bit级的操作函数。不出意外的是，这些函数都是与CPU架构相关的。它们在`<asm/bitops.h>`中声明。

这些函数的参数是一个指向被操作数的指针和一个整数用以表示需要操作的位。第0位bit是最低位。在32位机上，第31位bit是最高位比特，而第32位bit是下一个word的最低位。

需要注意的是：对于位的操作不必非要使用`atomic_t`类型的数据。指向任何类型的指针都可以作为这些函数的操作数，例如：

```C
unsigned long word = 0;
set_bit(0, &word);
```

内核除了提供原子性的位操作外，还提供了非原子性的位操作。这些非原子性的操作的唯一区别是函数名加了`__`两个下划线作为前缀。如果你的代码已经受到了其他的锁的保护，使用非原子性的位操作将会提升代码的执行速度。否则，请使用原子性的位操作。

# Spin Locks

在内核中，最常见的就是**spin lock**了。

**spin lock**仅仅能被一个运行中的线程所获得。如果一个线程尝试获取一个已经被占用的自旋锁，一般被称为**contended**，那么该线程会陷入无限循环中即**spin**，直到这个锁被释放。如果这个锁没有**contended**，那么该线程则可以立即获得该锁。

spin保证了在任何时候，不超过一个的可执行线程进入临界区。

由于自旋锁会以消耗CPU时间为代价来等待锁的释放，因此长时间的占用自旋锁是不明智的。一种替代的方法是，当某个线程需要获取的自旋锁处于**contended**状态时，则将该线程休眠，等待该锁被释放后，再唤醒该线程。然而这种方法会导致一些额外的开销，即两次上下文切换所造成的开销。这种开销已经远远超过了使用自旋锁带来的好处。

**因此，最明智的做法是将占用自旋锁的持续时间控制在小于两次进程的切换时间**。不过由于估计上下文切换时间是困难的，因此尽量保证获取自旋锁的时间尽量短才是正确的。同时由于现在的内核均支持抢占（preemptive），占用自旋锁的时间与系统调度延迟的时间相等。

## Spin Lock Methods

自旋锁的实现与CPU架构有关，同时实现它们的代码均为汇编。

与CPU架构有关的代码定义在`<asm/spinlock.h>`中。实际可用的接口函数定义在`<linux/spinlock.h>`中。

自旋锁的基本用法如下：

```C
DEFINE_SPINLOCK(mr_lock);
spin_lock(&mr_lock);
/* critical region ... */
spin_unlock(&mr_lock);
```

> 在单CPU的机器上，锁并既不会被被编译也不会存在，这里的锁仅仅是禁止了内核抢占。如果内核在编译时禁用了抢占，那么什么都不会存在了。

需要特别注意的是，内核的自旋锁不同于用户空间的线程库中的自旋锁，内核空间的自旋锁是不能递归调用的。在内核中递归的获取自旋锁会导致内核死锁。

**自旋锁不会引起进程休眠，因此可以合法的在中断处理函数中使用自旋锁。由于信号量会导致休眠，因此不能在中断处理函数中使用自旋锁。**

**如果要在中断处理函数中使用自旋锁，必须先关闭同一CPU的中断，否则，中断的嵌套会造成自旋锁嵌套，从而被死锁。**

内核提供了能够同时关闭中断和获取自旋锁的接口：

```C
DEFINE_SPINLOCK(mr_lock);

unsigned long flags;
spin_lock_irqsave(&mr_lock, flags);
/* critical region ... */
spin_unlock_irqrestore(&mr_lock, flags);
```

如果不想保存寄存器中的内容，可以使用如下函数：

```C
DEFINE_SPINLOCK(mr_lock);

spin_lock_irq(&mr_lock);
/* critical section ... */
spin_unlock_irq(&mr_lock);
```

与调试自旋锁有关的内核配置项：

1. CONFIGDEBUGSPINLOCK
2. CONFIGDEBUGLOCK_ALLOC

## 与Spin Lock有关的函数

| Method                       | Description                                                                                   |
| :--------------------------: | :-------------------------------------------------------------------------------------------- |
| `spin_lock()`                | Acquires given lock                                                                           |
| `spin_lock_irq()`            | Disables local interrupts and acquires given lock                                             |
| `spin_lock_irqsave()`        | Saves current state of local interrupts, disables local interrupts, and acquires given lock   |
| `spin_unlock()`              | Releases given lock                                                                           |
| `spin_unlock_irq()`          | Releases given lock and enables local interrupts                                              |
| `spin_unlock_irqrestore()`   | Releases given lock and restores local interrupts to given previous state                     |
| `spin_lock_init()`           | Dynamically initializes given `spinlock_t`                                                    |
| `spin_trylock()`             | Tries to acquire given lock; if unavailable, returns nonzero                                  |
| `spinislocked()`             | Returns nonzero if the given lock is currently acquired, otherwise it returns zero            |

## Spin Lock & Bottom Halves

函数`spin_lock_bh()`获取自旋锁并且禁止所有Bottom Halves。`spin_unlock_bh()`做相反的工作。

1. 由于Bottom Half会抢占进程上下文的执行。因此，如果数据在Bottom Half的进程上下文之间共享的话，则必须使用锁并且禁止Bottom Halves。
2. 由于中断服务函数会抢占Bottom Half，因此如果数据在中断处理函数和bottom half之间共享的话，必须要获得锁并且禁止中断。
3. 由于两个类型相同的tasklet永远不会同时运行。因此在相同类型的tasklet间共享的数据不需要保护。
4. 如果在两个不同类型的tasklet间共享数据的话，必须在访问bottom half中数据之前获得锁。由于在同一CPU上的tasklet不会互相抢占，因此并不需要禁止bottom halves。
5. 不管是不是相同类型的softirq，只要他们之间有数据的共享就必须被锁保护。由于即使是两个相同类型的softirq也可能在不同的CPU上同时运行。同一个CPU上的softirq并不会互相抢占，因此不需要禁止bottom halves。

## Reader-Writer Spin Locks

**Reader-Writer Spin Locks**为锁提供了读了的reader和writer两个变种版本。一个或多个reader能够同时获得**reader lock**。然而**writer lock**只能同时被一个writer获取。

**Reader-Writer Spin Locks**有时被称作共享/互斥锁（shared/exclusive locks）或并发/互斥锁（concurrent/exclusive locks）。

**Reader-Writer Spin Locks**的使用方法如下：

```C
DEFINE_RWLOCK(mr_rwlock);

/* For readers */
read_lock(&mr_rwlock);
/* critical section (read only) ... */
read_unlock(&mr_rwlock);

/* For writers */
write_lock(&mr_rwlock);
/* critical section (read and write) ... */ write_unlock(&mr_lock);
```

与**Reader-Writer Spin Locks**有关的函数：

| Method                      | Description                                                                                                         |
| :-------------------------: | :------------------------------------------------------------------------------------------------------------------ |
| `read_lock()`               | Acquires given lock for reading                                                                                     |
| `read_lock_irq()`           | Disables local interrupts and acquires given lock for reading                                                       |
| `read_lock_irqsave()`       | Saves the current state of local interrupts, disables local interrupts, and acquires the given lock for reading     |
| `read_unlock()`             | Releases given lock for reading                                                                                     |
| `read_unlock_irq()`         | Releases given lock and enables local interrupts                                                                    |
| `readunlock irqrestore()`   | Releases given lock and restores local interrupts to the given previous state                                       |
| `write_lock()`              | Acquires given lock for writing                                                                                     |
| `write_lock_irq()`          | Disables local interrupts and acquires the given lock for writing                                                   |
| `write_lock_irqsave()`      | Saves current state of local interrupts, disables local interrupts, and acquires the given lock for writing         |
| `write_unlock()`            | Releases given lock                                                                                                 |
| `write_unlock_irq()`        | Releases given lock and enables local interrupts                                                                    |
| `write_unlock_irqrestore()` | Releases given lock and restores local interrupts to given previous state                                           |
| `write_trylock()`           | Tries to acquire given lock for writing; if unavailable, returns nonzero                                            |
| `rwlock_init()`             | Initializes given `rwlock_t`                                                                                        |

对于**reader-writer lock**还需要注意的是，这种锁更偏向于reader，也就是说只要有一个reader获取改锁，那么就不会有writer获取。

## Spin Lock总结
自旋锁提供了简单并且快速的的锁机制。自旋锁被优化为应对较短的占用时间同时，获取到自旋锁的代码不能休眠。如果在获取到锁后需要较长时间的休眠，那么信号量是一个好的解决方案。

# Semaphores (信号量)

信号量在Linux内核中是能够被**休眠**的锁。当某个任务尝试获得的信号量不能立刻满足时，系统会将该任务加入到一个等待队列中并且将改任务休眠。这样处理器就能够被释放从而执行其他的代码。

信号量具有如下的特点：

1. 由于在等待处于contended状态的信号量时，任务可以被休眠，因此信号量非常适合锁获取时间较长的场合。
2. 相反的，由于信号量没有被优化用以应对较短的锁时间，因此，如果占用锁的时间过短的话反而会增加整个系统的开销。
3. 由于代码的执行路线会因为信号量而被休眠，因此，获取信号量的代码只能存在于进程上下文中，而不能存在于中断上下文中。毕竟中断上下文是不能够被调度的。
4. 获得信号量的任务能够被休眠，因为信号量并不像自旋锁会一直占用CPU。当一个任务因为等待信号量而休眠时，其他的任务会被调度至CPU，从而避免了死锁的发生。
5. 在获取获得自旋锁后再获取信号量，这是因为信号量会使得任务被休眠，而这对自旋锁来说是不允许对。

> 如果你的代码经常需要休眠，信号量是唯一的解决方案。 通常在非必需的情况下，使用信号量会更容易，因为他们允许休眠。

如果在某个场景下，自旋锁和信号量都能胜任，那么具体的选择应该由占用锁的时间长短锁决定。

除此之外，和自旋锁不同的是，信号量**不会禁止内核的抢占**，因此占用信号量的任务能够被其他的任务所抢占。这也就意味着，信号量不会造成调度的延迟。

## Counting and Binary Semaphores

信号量的最后一个特点是允许同时被任意数量的任务占用。然而，自旋锁仅能允许被一个任务占用。信号量所能允许的最大占用数量是在声明的时候就决定的。这个值被称作**usage count**或者简称**count**。

如果将信号量的count设置为1，那么它就能被称作**binary semaphore**或**mutex**。同样的，**count**也能被初始化为任何大于1的非零数。在这种情况下的信号量被称作**counting semaphore**。

**counting semaphore**在内核中使用的很少，一般都使用**mutex**。

## Creating and Initializing Semaphore

和自旋锁一样，信号量的具体实现是与所使用的CPU架构有关的。所有有关信号量的定义均包含在**<asm/semaphore.h>**中。信号量使用**struct semaphore**数据结构来表示。

静态声明的信号量通过以下方法创建，其中**name**是该变量的名字，而**count**是该信号量的**usage count**：

```C
struct semaphore name;
sema_init(&name, count);
```

如果想快速的创建一个**mutex**，使用如下的方法：

```C
static DECLARE_MUTEX(name);
```

动态声明和创建信号量的方法如下：

```C
sema_init(sem, count);
init_MUTEX(sem);
```

## 信号量的使用

函数`down_interruptible()`会尝试获取指定的信号量。如果信号量不能立即获得，该函数会讲调用该函数的进程休眠，即转为`TASK_INTERRUPTIBLE`状态。如果任务在等待信号量可用的过程中收到了信号，那么`down_interruptible()`会返回`-EINTR`。

相似的`down()`函数会讲调用它的任务转为`TASK_UNINTERRUPTIBLE`状态。

如果要释放一个信号量需要使用`up()`函数。

下面是一个简单的例子：

```C
/* define and declare a semaphore, named mr_sem, with a count of one */
static DECLARE_MUTEX(mr_sem);

/* attempt to acquire the semaphore ... */ 
if (down_interruptible(&mr_sem)) {
    /* signal received, semaphore not acquired ... */
}

/* critical region ... */

/* release the given semaphore */ 
up(&mr_sem);
```

与信号量有关的全部操作如下表：

| Method                                    | Description                                                                                      |
| :---------------------------------------: | :----------------------------------------------------------------------------------------------- |
| `sema_init(struct semaphore, int)`        | Initializes the dynamically created semaphore to the given count                                 |
| `init_MUTEX(struct semaphore)`            | Initializes the dynamically created semaphore with a count of one                                |
| `init_MUTEX_LOCKED(struct semaphore)`     | Initializes the dynamically created semaphore with a count of zero (so it is initially locked)   |
| `down_interruptible (struct semaphore)`   | Tries to acquire the given semaphore and enter interruptible sleep if it is contended            |
| `down(struct semaphore)`                  | Tries to acquire the given semaphore and enter uninterruptible sleep if it is contended          |
| `down_trylock(struct semaphore)`          | Tries to acquire the given semaphore and immediately return nonzero if it is contended           |
| `up(struct semaphore *)`                  | Releases the given semaphore and wakes a waiting task, if any                                    |

## Reader-Writer Semaphores

信号量和自旋锁一样，拥有一套**Reader-Writer**类型的信号量。

Reader-Writer信号量用`struct rw_semaphore`类型表示，该类型在`<linux/rwsem.h>`文件中定义。静态和动态的初始化Reader-Writers Semaphore的方法如下：

```C
static DECLARE_RWSEM(name);
init_rwsem(struct rw_semaphore *sem);
```

**需要注意的是，所有的Reader-Writer信号量都是`mutex`，即他们的usage counting总是1。**

所有的Reader-Writer锁都是`uninterruptible`的，因此它们仅有一个`down()`函数。

```C
static DECLARE_RWSEM(mr_rwsem);

/* attempt to acquire the semaphore for reading ... */
down_read(&mr_rwsem);

/* critical region (read only) ... */

/* release the semaphore */
up_read(&mr_rwsem);

/* ... */

/* attempt to acquire the semaphore for writing ... */
down_write(&mr_rwsem);

/* critical region (read and write) ... */

/* release the semaphore */
up_write(&mr_sem);
```

和自旋锁相比，信号量的的Reader-Writer版本多了一个downgrade_write()函数。这个函数能够自动的将所获得的write lock转变为read lock。

# Mutex

由于用户经常将信号量用作**mutex**，即可以睡眠的自旋锁。由于信号量本身更加通用，同时也没有多少使用限制。这也使得信号量适合那些不明确的，复杂的互斥访问控制，比如内核空间和用户空间的负复杂交互行为。但是这也意味着信号量不能很容易的被使用且缺少某种自动调试方法。为了寻找一种简单的能够休眠的锁，内核开发者引入了**mutex**互斥体。

互斥体使用`struct mutex`结构来表示。它感觉上和信号量相似，但是互斥体却有一个更加容易使用的接口，更加的高效还有使用上的额外限制。

静态和动态定义互斥体的方法如下：

```C
/* statically define a mutex */
DEFINE_MUTEX(name);

/* dynamically define a mutex */ 
mutex_init(&mutex);
```

对互斥体的加锁和解锁非常的简单：

```C
mutex_lock(&mutex);
/* critical region ... */
mutex_unlock(&mutex);
```

下表是关于互斥体的相关方法：

| Method                             | Description                                                                                                 |
| :--------------------------------: | :---------------------------------------------------------------------------------------------------------- |
| `mutex_lock(struct mutex)`         | Locks the given mutex; sleeps if the lock is unavailable                                                    |
| `mutex_unlock(struct mutex)`       | Unlocks the given mutex                                                                                     |
| `mutex_trylock(struct mutex)`      | Tries to acquire the given mutex; returns one if suc- cessful and the lock is acquired and zero otherwise   |
| `mutex_is_locked (struct mutex)`   | Returns one if the lock is locked and zero otherwise                                                        |

任何事情都是有两面性的，在互斥体带来简单与方便的同时，它的使用也有很多额外的限制。互斥体相比信号量的使用更加的严格和狭窄：

- 只能由一个任务获取互斥体。
- 不管是谁获取了互斥体，它必须在使用完后解锁互斥体，即不能在一个上下文中获取互斥体，而在另一个上下文中解锁互斥体。这意味着互斥体不能用在类似内核－用户空间的同步上。
- 互斥体部允许的加锁和解锁是不允许递归的。
- 进程在获得互斥体后不能退出。
- **互斥体不能在中断处理函数和Bottom Halves中使用，哪怕是mutex_trylock()函数也不行。**
- 互斥体只能通过官方的API来管理。

配置内核开启CONFIG_DEBUG_MUTEXES编译选项，会有多种检测来确保这些约束得以遵守。

# Completion Variables

在内核中，当一个任务需要通知其他的任务某个事件出现时，使用Completion Variables是最简单的同步方式。一个任务等待completion variable，另一个任务进行其他某些工作。当另一个任务完成工作时，使用completion variable来唤醒其他的正在等待的任务。

或许，你会觉得这听起来和信号量的作用很像。事实上completion variable仅仅是为代替信号量提供了一个简单的解决方法。例如当子进程退出时，可以使用completion variable来唤醒父进程。

completion varibales使用`struct completion`类型来表示，该类型定义在`<linux/completion.h>`中。

静态和动态创建completion variables的方法如下：

```C
/* statically define a completion variable */
DECLARE_COMPLETION(mr_comp);

/* dynamically define a completion variable */
init_completion(&mr_comp);
```

给定一个completion variable，需要等待的任务需要调用`wait_for_completion()`。当等待的时间发生了之后，调用`complete()`函数唤醒所有的正在等待的任务。下表列出了有关completion varibale的相关方法：

| Method                                      | Description                                                     |
| :-----------------------------------------: | :-------------------------------------------------------------- |
| `init_completion(struct completion )`       | Initializes the given dynamically created completion variable   |
| `wait_for_completion(struct completion )`   | Waits for the given completion variable to be signaled          |
| `complete(struct completion *)`             | Signals any waiting tasks to wake up                            |

在`kernel/sched.c`和`kernel/fork.c`中有大量的关于**completion variable**的例子。一个常见的场景是，内核代码等待某个数据结构的初始化时调用`wait_for_completion()`，当初始化完成后，这些等待的任务会通过`completion()`来唤醒。

---

# BKL: The Big Kernel Lock

Big Kernel Lock最早提出是为了解决SMP兼容问题而加入的。它是一个全局的自旋锁。由于在最新的内核和驱动中已经不鼓励使用BKL，因此这里我们就不做深入的介绍了。

有兴趣的读者可以参考[Here](http://kernelnewbies.org/BigKernelLock)和[Here](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=4ba8216cd90560bc402f52076f64d8546e8aefcb)。

# Sequential Locks

**Sequential Lock**被简写为**seq lock**，它是在内核2.6中被引入的新类型。**seq lock**提供了一个简单的机制用以读写共享的数据。它的工作机制基于一个顺序计数器。在任何时候，有疑义的数据被写入的时候，就会获得一把锁，并且序列值会增加。在读取数据之前和之后，读取序列值。如果序列值相同的话，那么在读数据的时候，并没有写操作发生。更进一步将，如果序列值是偶数的话，说明当前没有写操作正在进行（由于初始值为0，写操作获得和释放锁时均会让序列值加1）。

定义一个**seq locks**使用如下的方法：

```C
seqlock_t mr_seq_lock = DEFINE_SEQLOCK(mr_seq_lock);
```

之后，使用**seq lock**保护数据写入的方法如下：

```C
write_seqlock(&mr_seq_lock);
/* write lock is obtained... */
write_sequnlock(&mr_seq_lock);
```

对于使用**seq lock**保护的写入，代码可能会看起来有点奇怪：

```C
do {
    seq = read_seqbegin(&mr_seq_lock);
    /* read data here ... */
} while (read_seqretry(&mr_seq_lock, seq));
```

和Reader-Writer信号量不同的是，**seq lock**更加倾向于写，而非读。通常使用**seq lock**保护的写入只要没有别的写入同时进行的话都会成功。对保护内容的读取并不会影响写入。进一步讲，写入操作总会造成读取操作被不停的循环，直到没有任何写入获取这个锁。

当你遇到如下的情形时，**seq lock**会非常的适合：

- 数据拥有非常多的读取者。
- 数据有很少的写入者。
- 虽然有很少的写入者，但是数据更倾向于写，并且不允许读取造成写入的饥饿。
- 数据非常的简单，例如一个简单的结构体，甚至简单的整数，但是因为某些原因，不能使用atomic变量。

---

### ¶ The end
