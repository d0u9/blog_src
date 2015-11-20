---
title: 'Ghost博客搭建'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
abbrlink: 2360490712
date: 2015-11-20 18:01:00
---

之前一直用Wordpress搭建我的个人博客，但是用了一段时间后总觉得Wordpress有些重，对于我这种只是用博客来记录一些日常见闻和游记的人来说，不太好用。纠结了许久，通过谷哥找到了Ghost这款轻量级的，静态化的博客框架，试用了一下发现还不错。遂将安装过程记录如下：

<!-- more -->

# 环境说明

我所用的系统为Ubuntu 14.04 X64版本。在一下的各步骤中，标记为`$`开头的为普通用户操作，标记为`#`的为root用户操作。

# 安装Mysql数据库

> 参考：<https://allaboutghost.com/migrating-ghost-installation-from-sqlite3-to-mysql/>

由于Ghost默认使用的是sqlite数据库，为了能够更好的提升性能，也为了以后迁移数据更加方便，在本文中，我们修改Ghost的默认数据库为Mysql数据库。

1. 更新系统：

   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. 安装Mysql数据库：

   ```bash
   sudo apt-get install mysql-server -y
   ```

3. 配置Mysql。所有选项默认即可：

   ```bash
   mysql_secure_installation
   ```

4. 登陆Mysql：

   ```bash
   mysql -u root -p
   ```

5. 为Ghost新建一个数据库，执行Mysql命令：

   ```sql
   create database ghost;
   ```

6. 为Ghost创建新的数据库用户，执行Mysql命令，<font color='red'>注意将其中password替换为自己的数据库密码</font>：

   ```sql
   CREATE USER 'ghost'@'localhost' IDENTIFIED BY 'password';
   ```

7. 为刚才创建的新用户赋予权限，执行Mysql命令：

   ```sql
   GRANT ALL PRIVILEGES ON ghost.* TO 'ghost'@'localhost';
   ```

8. 刷新权限，执行Mysql命令：

   ```sql
   flush privileges;
   ```

9. 退出Mysql，执行Mysql命令： 

   ```sql
   quit;
   ```
# 安装Nginx服务器

执行以下命令：

   ```sql
   sudo apt-get install -y nginx
   ```

# 安装Ghost博客框架

> 参考：<https://www.howtoinstallghost.com/how-to-install-ghost-on-ubuntu-server-12-04/>

1. 安装必要的软件包：

   ```bash
   sudo apt-get install -y zip vim wget
   ```

2. 安装Node.js：

   ```bash
   curl -sL https://deb.nodesource.com/setup | sudo bash -
   sudo apt-get install -y nodejs
   ```

3. 下载并安装Ghost：

   ```bash
   sudo mkdir -p /var/www/
   cd /var/www/
   sudo wget https://ghost.org/zip/ghost-latest.zip
   sudo unzip -d ghost ghost-latest.zip
   sudo rm ghost-latest.zip
   cd ghost/
   sudo npm install --production
   ```

# 配置Ghost使用Mysql

> 参考：<https://allaboutghost.com/migrating-ghost-installation-from-sqlite3-to-mysql/>

1. 在Ghost的目录中创建配置文件：

   ```bash
   cd /var/www/ghost
   sudo cp config.example.js config.js
   ```

2. 修改配置文件，使用任何你喜欢的编辑器编辑`config.js`,在`Production`这一节，修改如下内容：

   1) 设置本地域名，修改`url`到博客的域名。

   ```
   url: 'http://my-ghost-blog.com',
   ```

   2) 设置Ghost使用Mysql数据库，修改：

   ```
   database: {
       client: 'mysql',
       connection: {
           host: 'localhost',
           user: 'ghost',
           password: 'password',
           database: 'ghost',
           charset: 'utf8'
       },
       debug: true
   },
   ```

   将其中的`user`和`password`是我们之前在Mysql中创建的用户和对应的密码。`database`是我们之前创建的数据库名称。

   3) 让Ghost可以被外网访问，修改：

   ```
   host: '127.0.0.1',
   ```

   到

   ```
   host: '0.0.0.0',
   ```

3. 测试Ghost是否可以使用，执行

   ```bash
   sudo npm start --production
   ```

   访问`http://your-domain:2368`，看能否正常工作。

4. 如果可以工作的话，我们为了安全将：

   ```
   host: '0.0.0.0',
   ```

   改回

   ```
   host: '127.0.0.1',
   ```

# 配置PHP环境

> 参考：<https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04>

1. 安装PHP环境：

   ```bash
   sudo apt-get install -y php5-fpm php5-mysql
   ```

2. 配置PHP：

   编辑`/etc/php5/fpm/php.ini`文件，找到`cgi.fix_pathinfo`字段，并将其修改为：

   ```
   cgi.fix_pathinfo=0
   ```

3. 重启PHP:

   ```bash
   sudo service php5-fpm restart
   ```

# 配置Nginx虚拟服务器

## 配置Nginx反向代理Ghost

> 参考：<https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-14-04-lts>

为了让我们的PHP网站和Ghost博客不相互影响，需要先配置两个虚拟机，一个用来反向代理Nginx，另一个用来支持PHP。

1. 配置Ghost虚拟服务器，新建Nginx的Block文件：

   ```bash
   cd /etc/nginx/sites-available
   sudo cp default ghost
   ```

   编辑ghost文件内容如下：

   ```
   server {
       listen 80 default_server;
       listen [::]:80 default_server ipv6only=on;

       server_name your-domain;

       location / {
           proxy_set_header   X-Real-IP $remote_addr;
           proxy_set_header   Host      $http_host;
           proxy_pass         http://127.0.0.1:2368;
       }
   }
   ```

   将`server_name`修改为你的域名。

2. 让刚才的配置生效：

   ```bash
   sudo rm /etc/nginx/sites-enabled/default
   sudo ln -s /etc/nginx/sites-available/ghost /etc/nginx/sites-enabled
   sudo service nginx restart
   ```

3. 测试是否生效：

   1) 启动Ghost:

   ```bash
   cd /var/www/ghost
   sudo npm start --production
   ```

   2) 访问你的域名或IP看是否能够打开Ghost。

## 配置PHP虚拟服务器

1. 新建虚拟服务器文件夹：

   ```bash
   sudo mkdir -p /var/www/root/html
   ```

2. 新建Nginx的Block文件：

   ```bash
   cd /etc/nginx/sites-available
   sudo cp default root
   ```

   修改其内容如下：

   ```
   server {
       listen 8080 default_server;
       listen [::]:8080 default_server ipv6only=on;

       root /var/www/root/html;
       index index.php index.html index.htm;

       server_name root;

       location / {
           try_files $uri $uri/ =404;
       }

       error_page 404 /404.html;
       error_page 500 502 503 504 /50x.html;
       location = /50x.html {
           root /usr/share/nginx/html;
       }

       location ~ \.php$ {
           try_files $uri =404;
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/var/run/php5-fpm.sock;
           fastcgi_index index.php;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }
   }
   ```

3. **让刚才的配置生效**：

   ```bash
   sudo ln -s /etc/nginx/sites-available/root /etc/nginx/sites-enabled
   sudo service nginx restart
   ```

4. 测试是否生效

   新建测试文件：

   ```bash
   sudo touch /var/www/root/html/index.php
   ```

   编辑该文件，内容如下：

   ```
   <?php
       echo "Hello World!";
       echo "<br />";
       echo date('Y - m - d');
   ?>
   ```

   访问你的网页：

   ```
   http://your-domain:8080
   ```

# 让你的Ghost永久服务

> 参考：<http://support.ghost.org/deploying-ghost/>

为了让我们的Ghost服务器不会在我们断开远程连接后关闭，我们需要进行如下设置：

1. 安装`forever`:

   ```bash
   sudo npm install forever -g
   ```

2. 新建启动脚本：

   ```bash
   sudo touch /var/www/ghost/start_ghost.sh
   ```

   在脚本中添加如下内容：

   ```
   #! /bin/sh

   NODE_ENV=production forever start /var/www/ghost/index.js
   ```

   赋予可执行权限：

    ```bash
    sudo chmod -R 755 /var/www/ghost/start_ghost.sh
    ```

3. 测试运行：

   ```bash
   su
   /var/www/ghost/start_ghost.sh
   ```

   访问你的域名，看能否打开。

   使用`forever list`可以查看当前运行的任务，使用`forever stop /var/www/ghost/index.js` 可以停止Ghost服务。

# 设置开机启动

1. 在`/etc/rc.local`文件中，`exit 0`前添加：

   ```
   /var/www/ghost/start_ghost.sh
   ```

2. 重启机器并测试。

# 配置Ghost

1. 配置邮件服务。邮件服务可以赋予你的Ghost发邮件的能力。

   > 参考：<http://support.ghost.org/mail/>

   这里我使用的是新注册的Gmail账户。

   编辑`/var/www/ghost/config.js`文件，修改其中的`mail`部分：

   ```
   mail: {
       transport: 'SMTP',
       options: {
           service: 'Gmail',
           auth: {
               user: 'youremail@gmail.com',
               pass: 'yourpassword'
           }
       }
   },
   ```

   为了安全，Gmail的密码我使用的是两步验证生成的App密码。

2. 配置Ghost用户。访问：

   ```
   http://your-domain/ghost/
   ```

   可以打开设置页面。第一次打开该页面可以注册管理员账号。

---

### ¶ The end 

