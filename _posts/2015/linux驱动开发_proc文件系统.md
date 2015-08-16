---
title: 'Linux驱动开发--proc文件系统'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 2901188138
date: 2015-08-16 08:32:00
---

**/proc**是内核导出的一个虚拟文件系统，它并不实际的存在于此盘中，而是由系统通过文件模型模拟出来。**/proc**的主要用途是用来于内核进行交互，因此在**/proc**文件夹中，我们可以查看到很多于系统相关的信息。例如**/proc/modules**中包括了所有的当前已加载模块的列表，**/proc/meminfo**中列出了当前系统中内存使用状况的信息。除此之外，在驱动模块的开发中，**/proc**文件系统往往扮演着调试输出的角色。我们可以将想要查询的状态防到**/proc**中用以时实的查询。

<!-- more -->

# **/proc**文件系统简介

下面的长难句出自[维基百科](http://en.wikipedia.org/wiki/Procfs)。

> procfs (or the proc filesystem) is a special filesystem in Unix-like operating systems that presents information about processes and other system information in a hierarchical file-like structure, providing a more convenient and standardized method for dynamically accessing process data held in the kernel than traditional tracing methods or direct access to kernel memory.

# 在驱动中使用/proc文件系统

## 用到的函数和头文件

与**/proc**相关的头文件分别为:

```C
#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
```

有关**/proc**文件系统的函数定义可以在源码树的`include/linux/proc_fs.h`中找到。

```C
/**
 * proc_mkdir - 在proc文件系统中创建一个文件夹。
 * @name: 要创建的文件夹名
 * @parent: 要创建文件夹的父目录
 */
struct proc_dir_entry *proc_mkdir(const char *name, struct proc_dir_entry *parent)

/**
 * proc_create - 在proc文件系统中创建一个文件
 * @name: 要创建的文件名
 * @mode: 文件的权限
 * @parent: 文件所属父文件夹的结构
 * @proc_fops: 与该/proc文件相关的操作
 */
struct proc_dir_entry *proc_create(
       const char *name, umode_t mode, struct proc_dir_entry *parent,
       const struct file_operations *proc_fops)

/**
 * remove_proc_entry - 从/proc文件系统移除一个文件或者文件夹
 * @name: 文件或文件夹的名称
 * @parent: 该文件或文件夹所属的父目录，如果父目录为/proc，则该参数为NULL
 */
void remove_proc_entry(const char *name, struct proc_dir_entry *parent)

/**
 * single_open - 通过调用制定函数将内容写至`/proc`文件系统中。
 * @file: 打开文件的描述结构
 * @show: 函数指针，该函数会将需要的内容输出
 * @data: 指针，可以指向任何你希望的地址。用法与file->private_data类似。
 */
int single_open(struct file *file, int (*show)(struct seq_file *, void *),
                 void *data)

/**
 * single_release - 释放使用`single_open`打开的文件描述结构内存
 * @inode: 文件的inode描述
 * @file: 打开文件的描述结构
 */
int single_release(struct inode *inode, struct file *file)
```

除此之外，还有其他的函数，我们下面用到的时候再说其用法。

## 模块的加载和卸载函数
在模块的加载函数中，我们首先创建一个名叫**my_sub_dir**的文件夹，之后再在其中创建一个名叫**proc-fs**的文件用以输出我们需要的信息。

其代码如下：

```C
#define SUB_DIR_NAME    "my_sub_dir"
#define PROC_FS_NAME    "proc-fs"

static int __init start_module(void)
{
    PDEBUG("\n\tModule 'proc-fs' is loaded\n");

    parent = proc_mkdir(SUB_DIR_NAME, NULL);
    if (!proc_create(PROC_FS_NAME, 0, parent, &proc_ops))
        return -ENOMEM;

    return 0;
}
```

如果你想直接在**/proc**文件夹中创建文件，而不是某个字文件夹的话，只需将`proc_create()`函数的第三个参数传入`NULL`即可。

在模块卸载的时候，我们需要将我们创建的这两个文件删除，删除文件相对比较简单，只需调用`remove_proc_entry()`函数。

代码如下：

```C
static void __exit end_module(void)
{
    PDEBUG("Module 'proc-fs' is unloaded\n");

    remove_proc_entry(PROC_FS_NAME, parent);
    remove_proc_entry(SUB_DIR_NAME, NULL);
}
```

## **/proc**文件系统的读写操作

和普通文件读写一样，我们需要创建一个`struct file_operations`结构，并将其和我们创建的**/proc**文件绑定。

创建`struct file_operations`结构的代码：

```C
struct file_operations proc_ops = {
    .owner = THIS_MODULE,
    .open = proc_open,
    .read = seq_read,
    .llseek = seq_lseek,
    .release = proc_release,
};
```

在这个结构中，`seq_read()`和`seq_lseek()`是已经由内核定义好的，而`proc_release()`只是对内核提供的`seq_release()`函数的再封装：

```C
static int proc_release(struct inode *inode, struct file *filp)
{
    return seq_release(inode, filp);
}
```

下面我们说说`proc_open()`这个函数。在本文中，我们的目的是实现一个非常基本的**proc**文件系统，因此，并没有使用**seq**的迭代器（可见*LDD 3rd ed.*的87页*The seq_file interface*节），我们在下一篇文章中会讨论**seq**迭代器的基本用法。在我们这里由于不使用**seq**迭代器，因此只需要简单的调用`single_open()`来绑定一个输出函数即可，在输出函数中，我们调用`seq_printf()`来打印一行`Hello World!`。

`proc_open()`如下：

```C
static int proc_show(struct seq_file *m, void *)
{
    seq_printf(m, "Hello World\n");
    return 0;
}

static int proc_open(struct inode *inode, struct file *filp)
{
    return single_open(filp, proc_show, NULL);
    return 0;
}
```

在`proc_show()`函数中，输出不能使用`printk()`函数，而是要使用针对**seq_file**输出的一组专用函数，具体如下：

```C
int seq_printf(struct seq_file *sfile, const char *fmt, ...);

int seq_putc(struct seq_file *sfile, char c);

int seq_puts(struct seq_file *sfile, const char *s);

int seq_escape(struct seq_file *m, const char *s, const char *esc);

int seq_path(struct seq_file *sfile, struct vfsmount *m, struct dentry
  *dentry, char *esc);
```

这些函数的介绍可以参考[LDD3](http://www.makelinux.net/ldd3/chp-4-sect-3)的*4.3.1.4 The seq_file interface*节。

## 完整版代码

完整的代码可以参考[我的Github主页](https://github.com/d0u9/Linux-Device-Driver/tree/master/04_proc_fs_basic)。

---

### ¶ The end

