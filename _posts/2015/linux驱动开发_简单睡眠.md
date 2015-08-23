---
title: 'Linux驱动开发--简单睡眠'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 1614808257
date: 2015-08-23 08:50:00
---

有时，我们的设备在应用程序请求时，数据并没有准备好，这个时候应该怎么办呢？考虑如下的情形：当应用程序通过socket发送TCP数据时，我们需要先建立连接，在连接没有建立好时，应用程序要么选择等待（同步IO）要么选择暂时放弃，等会儿再来检查（异步IO）。在同步IO状态下，进程一般会被挂起，知道TCP连接已经建立成功，这事应用程序才会被唤醒。

那么我们怎么样让某个应用程序进入睡眠状态，又怎么在数据驱动准备好时，唤醒等待的进程呢？

<!-- more -->

# 驱动介绍

在本文中，我们会实现一个简单的管道实例`pipe`设备。该设备内部包括了一个简单的环形缓冲区，当缓冲区为空时，任何读取该设备的进程会被置入睡眠状态，直到其他的进程为该设备写入数据。反之，当缓冲区满时，任何向该设备写入数据的进程将被挂起，直到有读取进程取出数据才会被唤醒。

本文仅仅使用了基本的睡眠函数，在下一篇文章中我会介绍其他睡眠的高级用法。

该驱动的完整代码可以在我的github中找到：[https://github.com/Douglas-Su/Linux-Device-Driver-3.18/tree/master/8_pipe-simple-sleep](https://github.com/Douglas-Su/Linux-Device-Driver-3.18/tree/master/8_pipe-simple-sleep)

# 用到的函数

有关睡眠的函数都定义在了`<linux/wait.h>`中。在简单睡眠中我们会用到如下的函数：

```C
/**
 * 静态的定义一个等待队列
 */
DECLARE_WAIT_QUEUE_HEAD(name);

/**
 * 动态的定义一个等待队列
 */
wait_queue_head_t my_queue;
init_waitqueue_head(&my_queue);

/**
 * 唤醒指定等待队列的所有进程
 */
wait_event(queue, condition)
wait_event_interruptible(queue, condition)
wait_event_timeout(queue, condition, timeout)
wait_event_interruptible_timeout(queue, condition, timeout)
```

这里主要说下有关唤醒的函数。

这些唤醒函数中，`queue`是等待队列的头。`condition`是任意一个布尔表达式，上面的这些函数在休眠结束后都要对该表达式求职；在条件为真之前，进程会保持休眠。由于该表达式可能会被多次求值，因此对该表达式的求值被要求不能带来任何副作用。

这些唤醒的函数在调用时，会唤醒指定队列中的所有进程。前两个函数的区别主要是可中断和不可中断。后两个函数在前两个函数的基础上加入了超时机制，其超时的参数的单位是`jiffy`值。到超时发生时，这两个函数都会返回`0`，而不管`condition`如何求值。

# 关键代码解析

由于唤醒是对指定队列中所有的进程进行唤醒，因此我们需要小心的控制唤醒过程，避免产生竞争。

先看代码

```C
static ssize_t pipe_write(struct file *file, const char __user * buff,
              size_t count, loff_t * f_pos)
{
    struct buff_t *pipe = file->private_data;
    if (mutex_lock_interruptible(&pipe->mutex))
        return -ERESTARTSYS;

    while (pipe->size == BUFF_SIZE) {
        mutex_unlock(&pipe->mutex);
        if (file->f_flags & O_NONBLOCK)
            return -EAGAIN;

        PDEBUG("%s(%d) is going to sleep when writing\n", current->comm,
               current->pid);
        if (wait_event_interruptible
            (pipe->write_queue, (pipe->size != BUFF_SIZE)))
            return -ERESTARTSYS;

        if (mutex_lock_interruptible(&pipe->mutex))
            return -ERESTARTSYS;
    }

    if (pipe->write_pos >= pipe->read_pos)
        count = min(count, (size_t) (BUFF_SIZE - pipe->write_pos));
    else
        count = min(count, (size_t) (pipe->read_pos - pipe->write_pos));

    PDEBUG("Prepare to write %li bytes\n", count);
    if (copy_from_user(pipe->buff + pipe->write_pos, buff, count)) {
        mutex_unlock(&pipe->mutex);
        return -EFAULT;
    }

    pipe->write_pos += count;
    pipe->size += count;
    if (pipe->write_pos == BUFF_SIZE)
        pipe->write_pos = 0;

    mutex_unlock(&pipe->mutex);
    wake_up_interruptible(&pipe->read_queue);

    PDEBUG("`write` finished, r_pos = %d, w_pos = %d\n", pipe->read_pos,
           pipe->write_pos);

    return count;
}
```

这是`write()`函数，`read()`函数与此类似。

需要特别注意的是8-21行的代码，这些代码主要是为了保证，每次仅有一个进程获得临界区（缓冲区）的访问权限。每次进程被唤醒后，需要先获得该缓冲区的锁，当获得锁之后进行缓冲区判断，若条件依然不满足的话，则再次睡眠。

需要注意的的是条件表达式可能会多次求值。

---

### ¶ The end
