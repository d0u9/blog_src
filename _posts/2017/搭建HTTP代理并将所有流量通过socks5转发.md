---
title: '搭建HTTP代理并将所有流量通过socks5转发'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - GFW
abbrlink: 1633999627
date: 2017-04-12 16:29:00
---

虽然目前有很多的工具或软件能够直接支持socks5代理，但是有一些软件并不能直接使用socks5代理，而仅支持http代理。

为了解决这里软件的代理问题，可以搭建一个http代理，并将代理的所有流量通过socks5来进行转发。

在这里我们使用的是**privoxy**工具，当然使用**polipo**工具也是可以的。

<!-- more -->

# 安装privoxy

Ubuntu 系统下：

```bash
sudo apt-get install privoxy
```

Arch Linux下：

```bash
sudo pacman -S privoxy
```

# 配置

默认情况下，privoxy使用`/etc/privoxy/config`作为配置文件，但用户也可以直接指定文件，语法为：

```
privoxy [options] config_file
```

这里假设我们socks5本地代理端口为1080，那么为了让privoxy将流量转发至socks5，需要在配置文件中添加如下信息：

```
listen-address  0.0.0.0:1081
forward-socks5 / localhost:1080 .
```

这里，privoxy监听HTTP代理的端口为1081。

# 启动privoxy代理

```
privoxy [config_file]
```

如果使用默认的`/etc/privoxy/config`配置文件，则可省略`[config_file]`参数。

---

### ¶ The end
