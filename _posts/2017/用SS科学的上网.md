---
title: '用SS科学的上网'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - GFW
abbrlink: 607723399
date: 2017-04-17 11:18:00
---

在一年前，写了一篇关于[如何上科学的网](../../posts/3049455864.html)的文章，使用的方法是直接在服务器上搭建SS服务端。在这一段时间中，[docker](https://www.docker.com/)技术得到了前所未有的发展，最近，我也尝试着将自己的SS服务进行了docker化，加速的服务器的部署速度。

<!-- more -->

# 服务端

## 前置条件

可以直接访问google.com等网站的服务器一台，并且需要有root用户权限。

## 搭建步骤

1. 安装[Docker](https://www.docker.com)

   这里以ubuntu 16.04操作系统为例说明安装方法：

   ```bash
   sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     software-properties-common
   ```

   添加gpg秘钥

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

   添加Docker源

   ```bash
   sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
   ```

   更新并安装Docker

   ```bash
   sudo apt-get update
   sudo apt-get -y install docker-ce
   ```

2. 下载我制作好的SS镜像

   ```bash
   docker pull d0u9/shadowsocks-libev

   具体的Dockerfile可以在[我的Github](https://github.com/d0u9/docker/tree/master/dockerfiles/shadowsocks-libev)看到。

3. 创建SS服务端配置文件

   SS的配置文件是一个json文件，在其中填写如下内容：

   ```json
   {
       "server":["[::0]", "0.0.0.0"],
       "server_port":8888,
       "password":"your_password",
       "timeout":360,
       "method":"aes-256-cfb",
   }
   ```

   - **server**: 监听地址，这里默认监听来自IPv4和IPv6的所有地址；

   - **server_port**: 服务监听端口;

   - **password**: SS的连接密码，用户必须提供该密码才能连接该SS服务；

   - **timeout**: 超时时间；

   - **method**: 加密方法；

   具体的配置说明可以参考Archlinux的wiki: [Shadowsocks](https://wiki.archlinux.org/index.php/Shadowsocks).

4. 创建容器并运行

   这里假设SS服务器配置文件路径为`path_to_ss_config`.

   ```bash
   docker run -d \
       --name <container_name> \
       -e CMD=server \
       -v <path_to_ss_config>:/ss.json:ro \
       -e SS_CONFIG_FILE=/ss.json \
       -p <ss_port>:<ss_port> \
       d0u9/shadowsocks-libev
   ```

   在上面这条命令中，所有的尖括号内容均需要用户自己按需配置。其中

   - **container\_name**: 该容器的名字；

   - **path\_to\_ss\_config**: 服务器的SS配置文件所在路径；

   - **ss\_port**: 在配置文件中指定的SS服务监听端口；

5. （可选）启动多个SS服务

   如果需要启用多个SS服务，只需运行多个docker容器即可，注意每个运行容器的名字必须唯一，监听端口要不同。

6. （可选）启用kcptun支持

   SS使用的TCP协议，TCP协议保证连接的可靠， 但也带来了较高的网络开销，例如各种拥塞控制和流量控制机制。为了减小这类开销，kcp这个基于udp的协议应运而生，具可以看体[这里](https://github.com/skywind3000/kcp)。

   kcp协议中数据的及时性靠发送大量相同内容的udp包来保证，而正确性是靠校验和算法来保证。

   [kcptun](https://github.com/xtaci/kcptun)项目是基于kcp协议的一个隧道。使用该工具可大幅降低SS的延时，但也会带来更多的带宽占用。

   在我的SS容器中，集成了kcptun工具，可以通过使用`-e enable_kcp=true`选项来开启。

   在启动kcptun工具的情况下，访问google.com的网络数据流为：

   ```
          TCP       TCP           UDP   |  UDP           TCP       TCP
   浏览器-----> SS -----> kcptun -----> | -----> kcptun -----> SS -----> google.com
   | ------------ 客户端 -------------- | -------------- 服务端 ------------------|
   ```

   kcptun同样使用json格式的文件来作为配置文件，简单的配置如下：

   对于客户端

   ```json
   {
       "remoteaddr": "kcp_ip:kcp_port",
       "localaddr": ":8888",
       "mode": "fast2",
       "MTU": 1400,
       "SndWnd": 256,
       "RcvWnd": 2048
   }
   ```

   - **kcp_ip:kcp_port**为运行kcp的服务器端地址和监听端口。

   - **localaddr**为本地监听端口，也就是说发到这个端口的数据都会被转换为kcp协议并发送给服务器。

   - **mode**为kcp工作的模式，一般用**fast2**即可。

   其余3个参数和网络调优有关，具体请参考[kcptun](https://github.com/xtaci/kcptun)文档。

   对于服务器端

   ```json
   {
       "target": "127.0.0.1:7863",
       "listen": ":7963",
       "mode": "fast2",
       "MTU": 1400,
       "SndWnd": 2048,
       "RcvWnd": 2048
   }
   ```

   - **target**指定的是将通过kcp协议收到的包，转发给哪个端口。这里填写的地址和端口是服务器端的SS监听的端口，由于kcp服务端和SS服务端都在一台机器中，地址写`127.0.0.1`就可以了。

   - **listen**是kcp服务监听的端口，kcp客户端会将UDP发送到这个端口。

   - **mode**和服务端一样，用**fast2**即可。

   其余参数和网络调优有关，具体请参考[kcptun](https://github.com/xtaci/kcptun)文档。

   在Docker中开启kcp协议：

   ```bash
   docker run -d \
       --name <container_name> \
       -e CMD=server \
       -v <path_to_ss_config>:/ss.json:ro \
       -e SS_CONFIG_FILE=/ss.json \
       -p <ss_port>:<ss_port> \
       -e ENABLE_KCP=true \
       -v <kcp_config_file>:/kcp.json \
       -e KCP_CONFIG_FILE=/kcp.json \
       d0u9/shadowsocks-libev
   ```

   根据你的系统，下载对应版本的kcptun，安装后，讲SS的服务端地址设置为本地，服务端口设置为kcptun客户端的监听端口即可。


# 客户端

SS支持多种不同的客户端。

## Windows

客户端项目主页: [https://github.com/shadowsocks/shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows).

## Linux， MAC OS

可以直接使用[shadowsocks-libev](https://githu.com/shadowsocks/shadowsocks-libev)，也可以使用docker配合我的shadowsocks-libev镜像。

## 使用Docker镜像

在我的Docker镜像中，可以开启客户端模式，这样就可以很容易的在不同的系统上运行SS客户端了，并且该方法的好处是可移植性很好，因为Docker帮我们屏蔽了底层系统。

Docker安装方法请一句自己的系统和官方文档进行安装。

下载我的Docker镜像：

```bash
docker pull d0u9/shadowsocks-libev
```

之后我们可以根据需要开启或不开启kcptun协议。

### 不开启kcptun协议

使用如下的命令创建并运行容器：

```
docker run -d \
    --name <container_name> \
    -e CMD=client \
    -e SS_CONFIG_FILE=/ss.json \
    -v <path_to_ss_config>:/ss.json:ro \
    -p <ss_port>:<ss_port> \
    d0u9/shadowsocks-libev
```

- **container\_name**: 该容器的名字；

- **path\_to\_ss\_config**: 服务器的SS配置文件所在路径；

- **ss\_port**: 在配置文件中指定的SS服务监听端口；

### 开启kcptun协议

使用如下的命令创建并运行容器：

```bash
docker run \
    --name <container_name> \
    -d \
    -e CMD=client \
    -e ENABLE_KCP=yes \
    -e SS_CONFIG_FILE=/ss.json \
    -v <path_to_ss_config>:/ss.json:ro \
    -e KCP_CONFIG_FILE=/kcp.json \
    -v <path_to_kcptun_config>:/kcp.json:ro \
    -p $ip:$port \
    d0u9/shadowsocks-libev
```

之后设置你的浏览器快乐的使用吧。

---

### ¶ The end
