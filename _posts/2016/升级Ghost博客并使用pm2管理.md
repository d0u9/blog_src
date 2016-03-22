---
title: 'Ghost博客搭建'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
abbrlink: 2360490714
date: 2016-03-22 12:28:00
---

最近对博客进行了升级，中间遇到了很多小问题。此外，升级后，不知为何`forever`不能自启动了，后来尝试了`pm2`发现很好用，遂用`pm2`代替了`forever`来管理Ghost博客。

<!-- more -->

# 升级Ghost博客

1. 下载最新版本的Ghost。

   ```bash
   curl -LOk https://ghost.org/zip/ghost-latest.zip
   ```

2. 解压Ghost到某个临时目录，这里假设临时目录为`ghost-temp`。

   ```bash
   unzip ghost-latest.zip -d ghost-temp
   ```

3. 删除原Ghost安装目录下的`core`，`index.js`，`node_modules`文件夹以及所有的json文件。

   Ghost一般默认的安装位置在`/var/www/ghost/`，这里使用`path-to-ghost-install`来代替。

   ```bash
   rm -fr path-to-ghost-install/core path-to-ghost-install/index.js path-to-ghost-install/node_modules path-to-ghost-install/*.json
   ```

4. 复制解压缩文件夹中的上述文件（node_moduels不包括）到原Ghost安装文件。

   ```bash
   cp -r ghost-temp/core ghost-temp/index.js ghost-temp/*.json path-to-ghost-install
   ```

5. 进入Ghost安装文件夹

   ```bash
   cd path-to-ghost-install
   ```

6. 安装Ghost

   ```bash
   npm install --production
   ```

   > 注意：如果你像我一样使用了DigitalOcean来托管Ghost，执行这一步可能会导致内存不足（DigitalOcean的虚拟机默认没有开启swap）,因此，可能需要在此处开启swap分区，保证安装成功，具体的方法见下一节。

7. 启动Ghost

   依据你使用的不同管理工具，可能会执行如下命令中的某一个`service restart ghost`，`forever restart index.js`，`pm2 restart ghosti`，`npm start --productionrs`。

# 为DigitalOcean开启swap

swap即虚拟内存，具体的概念请参考操作系统的相关书籍。简单的讲swap将磁盘的一部分虚拟成为了内存。

在DigitalOcean上的Linux中开启swap的方法如下：

1. 创建一个swap文件：

   ```bash
   sudo if=/dev/zero of=/swapfile bs=1024 count=1024k
   ```

   上述命令在根目录下创建了一个名为swap的文件，其大小为1024*1024k=1GB。

2. 建立Linux的swap区：

   ```bash
   sudo mkswap /swapfile
   ```

3. 使能上述swap区：

   ```bash
   sudo swapon /swapfile
   ```

# 使用pm2来管理Ghost

之前一直使用`forever`来管理Ghost，但这次升级之后，发现forever不能正常的开机子启动，遂切换至pm2。

1. 安装`pm2`：

   ```bash
   npm install pm2 -g
   ```

2. 使用`pm2`启动Ghost：

   ```bash
   NODE_ENV=production pm2 start index.js --name "Ghost"
   ```

3. 将Ghost添加至pm2的管理列表：

   ```bash
   pm2 save
   ```

4. 设置`pm2`开机自启动：

   ```bash
   sudo pm2 startup
   ```

   重启下系统，试试是否启动成功。


# 参考资料：

1. [http://support.ghost.org/deploying-ghost/](http://support.ghost.org/deploying-ghost/)
2. [http://pm2.keymetrics.io/docs/usage/startup/](http://pm2.keymetrics.io/docs/usage/startup/)
3. [http://support.ghost.org/how-to-upgrade/](http://support.ghost.org/how-to-upgrade/)

---

## ¶ The end


