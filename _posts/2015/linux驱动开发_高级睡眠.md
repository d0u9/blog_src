---
title: 'Linux驱动开发--高级睡眠'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 1381858328
date: 2015-08-26 10:54:00
---

在[Linux驱动开发--简单睡眠](../../posts/1614808257.html)中，我们介绍了如何将锁请求资源尚未准备好的进程转入睡眠状态，以及在条件满足时唤醒。但是由于我们在处理这类问题时往往需要面对更加复杂的情形，这时，简单睡眠就不能满足我们的需要了。为此在这里我们介绍高级睡眠模式。

本文中的示例和[Linux驱动开发--简单睡眠](../posts/16844.html)的示例基本相同，除了在处理睡眠时我们使用了更加底层的方式。

除此之外，本文中，我们还需要特别介绍**lost wake-up problem**。因为我在这里浪费了较多的时间。

<!-- more -->

# 进程睡眠

我们首先需要了解一个进程是怎么睡眠的，这个过程是怎么处理的。

进程可以具有`TASK_RUNNING`，`TASK_INTERRUPTIBLE`，`TASK_UNINTERRUPTIBLE`三种状态。当一个进程不具有`TASK_RUNNING`状态时，它不能被内核调度器调度到CPU上运行。更具体的讲，内核维护了一个所谓的`run queue`链表，链表中的每个节点都是一个具有`TASK_RUNNING`状态的进程。当一个进程的时间片被用完（或者其他情形）时，调度器会根据调度策略从`run queue`中选出一个认为最合适的来使用CPU。

除了当前占用CPU的进程（假设是进程A）时间片被用完时会触发调度之外，进程A也可以自愿的请求进程调度，让调度器选择下一个可以执行的进程。被选择的下一个进程可以是进程A也可以是其他的进程，这完全取决于调度算法和当前可运行进程的数量。进程自愿的请求调度通过调用`schedul()`函数来完成。

# 高级睡眠

## 用到的函数

```C
/**
 * 检查进程是否可以休眠，并将进程状态设置为state
 * @q: 等待队列的头
 * @wait: 将要插入的等待描述符
 * @state: 将进程修改为state状态
 */
void prepare_to_wait(wait_queue_head_t *q, wait_queue_t *wait, int state)

/**
 * 请求调度器选择下一个可以运行的进程
 */
void schedule(void)

/**
 * 修改进程状态为TASK_RUNNING，同时将该进程移出等待队列
 * @q: 等待队列的队列头
 * @wait: 当前的等待队列描述符
 */
void finish_wait(wait_queue_head_t *q, wait_queue_t *wait)
```

## 高级休眠实例

我们以`pipe_write()`为例讲解高级睡眠。在`pipe_write()`中有如下一段：

```C
while (pipe->valid_elements == MAX_BUFF_SIZE) {
    DEFINE_WAIT(wait);

    mutex_unlock(&pipe->mutex);
    if (filp->f_flags & O_NONBLOCK)
        return -EAGAIN;

    prepare_to_wait(&pipe->write_q, &wait, TASK_INTERRUPTIBLE);
    if (pipe->valid_elements == MAX_BUFF_SIZE)
        schedule();
    finish_wait(&pipe->write_q, &wait);

    if (signal_pending(current))
        return -ERESTARTSYS;
    if (mutex_lock_interruptible(&pipe->mutex))
        return -ERESTARTSYS;
}
```

首先我们需要一个循环用来检查我们等待的状态是否满足，也就是第1行的`while`循环。如果睡眠的条件在此处不满足，则直接跳过，执行后面的代码。如果睡眠条件满足，进入循环内部。需要注意的是，在执行这段代码之前我们已经获得了`pipe->mutex`。

在循环内部我们需要准备将当前进行写操作的进程转入睡眠状态。第一步是定义等待描述符号。在这里我们使用`DEFINE_WAIT(wait)`来静态的定义等待描述符。如果需要动态的定义等待描述符可以使用如下的代码：

```C
wait_queue_t wait;
init_wait(&wait);
```

此时我们需要释放已获得的锁，避免睡眠时拥有锁造成读取操作无法完成从而造成死锁。

在第8行，我们使用`prepare_to_wait()`将当前进程的加入等待队列中，同时将其状态变为`TASK_INTERRUPTIBLE`。需要明白的是，我们这时还没有放弃CPU的占用，但已经被加入了一个睡眠等待队列中。

之后我们在次检查唤醒状态是否满足，如果这时状态满足的话（可能是其他CPU上的进程完成了任务导致任务满足），则直接跳过`schedule()`执行`finish_wait()`。`finish_wait()`会讲进程的状态重新修改为`TASK_RUNNING`并且将其移出等待队列。如果条件不满足的话，则会执行`schedule()`，主动放弃CPU。直到被其他的进程唤醒。

## lost wake-up问题

我们将上述代码改为如下的情形：

```C
while (pipe->valid_elements == MAX_BUFF_SIZE) {
    DEFINE_WAIT(wait);

    mutex_unlock(&pipe->mutex);
    if (filp->f_flags & O_NONBLOCK)
        return -EAGAIN;

    if (pipe->valid_elements == MAX_BUFF_SIZE) {
        prepare_to_wait(&pipe->write_q, &wait, TASK_INTERRUPTIBLE);
        schedule();
    }

    finish_wait(&pipe->write_q, &wait);

    if (signal_pending(current))
        return -ERESTARTSYS;
    if (mutex_lock_interruptible(&pipe->mutex))
        return -ERESTARTSYS;
}
```

注意8、9行的改动。在这种情况下，如果有读取进程将缓冲区数据取走的时间正好发生在8行`if()`语句之后的话，那么我们的`write()`进程将无法感知到，从而继续进入睡眠状态。这也就是所谓的**lost wake-up**问题。

为什么我们的代码能避免**lost wake-up**问题呢？因为`prepare_to_wait()`函数会将当前的进程加入到等待队列中。此时进程不管何时都能被其他进程唤醒。

# 完整代码

完整的代码可以在我的github中找到：[https://github.com/d0u9/Linux-Device-Driver/tree/master/09_pipe_advanced_sleep](https://github.com/d0u9/Linux-Device-Driver/tree/master/09_pipe_advanced_sleep)

---

### ¶ The end

