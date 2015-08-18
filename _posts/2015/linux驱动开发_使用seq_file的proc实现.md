---
title: 'Linux驱动开发--使用seq_file的proc实现'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 403755856
date: 2015-08-18 16:30:00
---

由于`int single_open()`函数提供的功能相对较为简单，在需要输出较多数据的情况下，实现起来比较复杂。

为了让内核开发工作更加容易，通过对**/proc**代码的整理而增加了**seq_file**接口。这一接口为大的内核虚拟文件提供了一组简单的函数。

<!-- more -->

# seq_file介绍

**seq_file**本质上其实是一个迭代器，因此在使用的时候，首先需要先建立四个迭代器对象，其分别为`start()`, `next()`, `stop()`, `show()`。这四个迭代器会用来初始化结构`struct seq_operations`中的四个对应字段。

我们将**/proc**的`open()`方法简单的替换为`seq_open()`方法。`seq_open()`会被传入一个`struct seq_operations`结构，其中包含了迭代器的四个对象。这样后续的工作就可以由`seq_file`来接管了。

**seq_file**的具体工作流程如下图：

![](http://oyui6c341.bkt.clouddn.com/images/2015/linux驱动_使用seq_file的proc实现/01.png)

在`seq_file`工作的开始，会首先调用`start()`对象，该对象会完成一些初始化，之后整个`seq_file`会根据`start()`对象的返回值决定是否结束。如果`start()`返回的不是`NULL`，则反复继续调用`next()`方法，直到`next()`返回`NULL`退出循环，最后会调用`stop()`方法来进行一些收尾工作。之后再次进行上述流程。

**这里一定要注意的是，整个流程的唯一出口是由`start()`的返回值决定的，而非`stop()`。博主在这里折腾了很久，最终才注意到这个问题。**

在｀start()｀和｀stop()｀返回不为｀NULL｀时，会调用｀show()方法，将具体的数据输出。

**`seq_file`能够保证在`start()`方法和`stop()`方法之间不会有非原子的操作。我们可以确信，`start()`被调用之后会马上调用`stop()`。**

# 用到的函数和头文件

需要包含的头文件如下：

```C
#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
```

有关**/proc**文件系统的函数定义可以在源码树的**include/linux/proc_fs.h**中找到。

四个迭代器的原型如下：

```C
/**
 * start - 初始seq_file
 * @sfile: 表明当前使用的seq_file的结构体
 * @pos: 当前处理的位置
 *
 * start方法用以初始化seq_file的相关操作，如果该函数返回NULL，则整个过程结束。
 * *pos不必非要按照字节计算，例如，*pos可以用以表示当前读到数组的某一个项。
 * start方法的返回值会在其他的对象中作为输入函数，因此你可以返回一个数据在后续使用，例如链表当前节点的地址。
 */
void *start(struct seq_file *sfile, loff_t *pos);

/**
 * next - 迭代
 * @sfile: 表明当前使用的seq_file的结构体
 * @v: 由上一次next或start的返回值
 * @pos: 处理位置
 */
void *next(struct seq_file *sfile, void *v, loff_t *pos);

/**
 * @sfile: 表明当前使用的seq_file的结构体
 * @v: 由上一次next或start的返回值
 */
void stop(struct seq_file *sfile, void *v);

/**
 * show - 将内容输出
 * @sfile: 表明当前使用的seq_file的结构体
 * @v: 由上一次next或start的返回值
 *
 * 由`seq_printf()`系列函数将最后的结构输出。
 */
int show(struct seq_file *sfile, void *v);
```

# 一个示例

在这个示例中，我们将完成一个叫做**proc-fs-iterator.ko**的模块。模块加载完成后会在**/proc**中创建一个叫做**proc-fs-iterator**的文件。这个模块内部使用内核提供的双链表来完成输入的存储。在设备刚加载的时候，链表没有任何的节点。每次我们向**/proc/proc-fs-iterator**文件写入数据的时候，写入的数据会保存在链表的一个节点中。

当从该文件读出数据的时候，我们使用**seq_file**的迭代器完成每一个链表节点的遍历，从而输出所有的数据。

在此我们只对主要的函数进行说明，完整的函数可以在[我的Github上找到](https://github.com/d0u9/Linux-Device-Driver/tree/master/05_proc_fs_iterator)。

## 从/proc文件中读取数据。

其实从一个**/proc**文件中读取数据相对来说是稍微复杂一点的。其具体的步骤是将`/proc`文件当作一个普通的设备文件，在`struct file_operations`中绑定我们自己定义的`write()`函数，从而完成数据的读写。

在此，我们每次写的写操作均会创造一个新的链表节点，从而保存我们的数据。需要注意的是在这里，并非每次**open**创建新的节点，而是**write**。

函数如下：

```C
/*
 * WARNING: For the purpose of simplifying codes, we have no locks here to
 * pretect list head from concurrent access. So, obviously, it is unsafe.
 */
ssize_t proc_write(struct file *filp, const char __user *buff, size_t count,
    loff_t *f_pos)
{
    struct store_node *cur_node = NULL;
    PDEBUG("Write function is invoked\n");

    cur_node = (struct store_node *) kmalloc(sizeof(*cur_node), GFP_KERNEL);
    if (!cur_node)
        return -ENOMEM;

    if (!(cur_node->buf = kmalloc(count + 1, GFP_KERNEL))) {
        kfree(cur_node);
        return -ENOMEM;
    }
    memset(cur_node->buf, 0, count + 1);

    if (copy_from_user(cur_node->buf, buff, count)) {
        kfree(cur_node->buf);
        kfree(cur_node);
        return -EFAULT;
    }

    INIT_LIST_HEAD(&cur_node->list);
    list_add_tail(&cur_node->list, &store_list_head.list);
    atomic_inc(&store_list_head.n);

    PDEBUG("write %zd bytes, data is: '%s'\n", count, cur_node->buf);
    PDEBUG("Write function has finished\n");
    return count;
}
```

这里要注意的是，为了简化我们的函数，整个链表并没有用任何锁机制来保护，这也就是说，在两个进程同时写入的时候会产生竞争，从而造成数据的不完整或者链表的奔溃。

# 用seq_file来迭代的读取/proc/文件

## **/proc**的文件操作对象

和任何文件一样，我们的**/proc**文件也需要一组自定义的操作，这些操作应该同样的在`struct file_operations`中说明：

```C
struct file_operations proc_fops = {
    .owner = THIS_MODULE,
    .write = proc_write,
    .open = proc_seq_open,
    .read = seq_read,
    .llseek = seq_lseek,
    .release = seq_release,
};
```

这里的`proc_write()`函数是我们在*0x03:0x01*节中定义的函数。`seq_read()`，`seq_lseek()`，`seq_release()`是**seq_file**库的内置函数，他们自动帮我们完成外部对**proc**文件的读取任务，这里我们需要定义的是`proc_seq_open()`函数。

## `proc_seq_open()`函数

这个函数的主要任务是作为标准文件的`open()`函数和`seq_open()`函数的桥梁。`seq_open()`函数的原型如下：

```C
int seq_open(struct inode *inode, struct seq_operations *seq_ops);
```

第一个参数是标准`open()`函数传入的`struct inode`结构的地址。第二个函数是一个结构，其中的指针指向了我们自定义的四个seq_file迭代器对象。

```C
static struct seq_operations proc_seq_ops = {
    .start = proc_seq_start,
    .next = proc_seq_next,
    .stop = proc_seq_stop,
    .show = proc_seq_show,
};
```

其中的`proc_seq_start()`，`proc_seq_next()`，`proc_seq_stop()`，`proc_seq_show()`分别为我们自己定义的四个迭代器对象。

最后我们的`proc_seq_open()`函数如下：

```C
int proc_seq_open(struct inode *inode, struct file *filp)
{
    return seq_open(filp, &proc_seq_ops);
}
```

它只是我们对`seq_open()`函数的一个封装。

## `start`迭代器对象

`start`对象的主要作用时判断是否要继续迭代。当该函数返回`NULL`的时候，则停止迭代。需要注意的是，`start`才是整个`seq_file`的唯一退出位置，而非`stop`对象。

我们的`start`对象定义如下：

```C
static void *proc_seq_start(struct seq_file *s_file, loff_t *pos)
{
    PDEBUG("seq file start\n");
    if (*pos >= atomic_read(&store_list_head.n))
        return NULL;
    if (list_empty(&store_list_head.list))
        return NULL;
    return list_first_entry(&store_list_head.list, struct store_node, list);
}
```

其中的退出条件为：

1. 链表为空时
2. 已经遍历完了所有的链表节点

## `next`迭代器对象

`next`对象的作用是当上一次的`next`返回不为`NULL`时，继续进行迭代。参数表中的`void *`类型指针作为输入参数，其值时上一次`next`或`start`返回的地址，这个地址可以用作我们自己的参数。

具体实现如下：

```C
static void *proc_seq_next(struct seq_file *s_file, void *v, loff_t *pos)
{
    struct store_node *tmp = NULL;
    PDEBUG("seq file next\n");
    PDEBUG("operate at pos: %lld\n", *pos);
    (*pos)++;
    tmp = list_next_entry((struct store_node *)v, list);
    if (&tmp->list == &store_list_head.list) {
        PDEBUG("seq next will return NULL\n");
        return NULL;
    }

    PDEBUG("seq file now is returning %p\n", tmp);
    return tmp;
}
```

## `stop`迭代器对象

当`next`返回`NULL`时，会调用`stop`迭代器对象，该迭代器对象的主要目的是进行某些资源的释放，在这里我们没有什么需要释放的资源，因此该函数为空。

```C
static void proc_seq_stop(struct seq_file *s_file, void *v)
{
    PDEBUG("seq stop\n");
}
```

### `show`迭代器对象

在`start`和`next`返回不为`NULL`的时候，`show`会被调用，在`show`中可以使用`seq_printf()`系列的函数对变量内容进行输出。 具体可以参考这篇文章的2.3节最后部分。

# 全部的代码

见[https://github.com/d0u9/Linux-Device-Driver/tree/master/05_proc_fs_iterator](https://github.com/d0u9/Linux-Device-Driver/tree/master/05_proc_fs_iterator).

---

### ¶ The end

