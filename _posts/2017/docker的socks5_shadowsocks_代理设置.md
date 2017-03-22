---
title: 'docker的socks5(shadowsocks)代理设置'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - GFW
abbrlink: 449287862
date: 2017-03-22 20:13:00
---

docker在pull镜像时，默认是从官方Docker Hub下载。但由于某些原因，直接从官方Docker Hub拉去镜像时速度极慢或根本无法访问。通过使用国内的Docker Hub镜像站，例如阿里云的Docker镜像，可以加速访问。但对于有socks5代理的用户来讲，让docker 直接通过socks5代理，从官方Docker Hub来获取镜像时一种更好的方法。

<!-- more -->

**本说明仅支持使用systemd管理daemon进程的Linux系统**

# 步骤

1. 创建override文件：

   ```bash
   mkdir /etc/systemd/system/docker.service.d
   touch /etc/systemd/system/docker.service.d/all-proxy.conf
   ```

2. 编辑override文件：

   ```bash
   [Service]
   Environment="ALL_PROXY=socks5://localhost:1080/"
   ```

3. 重新加载配置，并重启docker服务：

   ```bash
   systemctl daemon-reload
   systemctl restart docker
   ```

# 参考文档

1. [http://stackoverflow.com/a/28093517/3824053](http://stackoverflow.com/a/28093517/3824053)
2. [https://docs.docker.com/engine/admin/systemd/](https://docs.docker.com/engine/admin/systemd/)

---

### ¶ The end
