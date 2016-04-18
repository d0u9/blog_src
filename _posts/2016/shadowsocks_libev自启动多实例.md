---
title: 'shadowsocks-libev自启动多实例'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - GFW
abbrlink: 4099256706
date: 2016-04-18 20:07:00
---

C语言编写的shadowsocks客户端/服务端软件shadowsocks-libev并不像go版本或python版本的shadowsocks客户端/服务端软件那样直接支持多实例配置（相关说明看[这里](https://github.com/shadowsocks/shadowsocks-libev/issues/5)）。但shadowsocks-libev由于使用纯C编写，其运行硬件要求较低，运行内存占用少，能够支持大并发等特点，很适合用于路由器或对性能要求敏感的用户。

最近有个需求：分配两个端口，两个端口采用不同的访问密码，分开管理。这可以通过采用启动多个ss-server进程实例来实现，每个进程实例使用不同的配置文件。

本文的方法是在默认shadowsocks-libev启动脚本的基础上进行修改实现的，方法比较粗糙。

<!-- more -->

1. 执行如下命令：

   ```bash
   cd /etc/shadowsocks-libev
   cp config.json config-demo.json
   ```

   可以将`config-demo.json`替换成任何你喜欢的名字。

2. 编辑`/etc/default/shadowsocks-libev`，在`Configuration file`部分添加：

   ```
   CONFFILE\_FAST\_OPEN="/etc/shadowsocks-libev/config-demo.json"
   ```

   这里的`config-demo.json`是在第一步中创建的新配置文件的名字。

3. 编辑shadowsocks-libev的启动文件`/etc/init.d/shadowsocks-libev`，修改为以下内容：

   ```bash
   #!/bin/sh
   ### BEGIN INIT INFO
   # Provides:          shadowsocks-libev
   # Required-Start:    $network $local_fs $remote_fs
   # Required-Stop:     $remote_fs
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: lightweight secured socks5 proxy
   # Description:       Shadowsocks-libev is a lightweight secured 
   #                    socks5 proxy for embedded devices and low end boxes.
   #
   ### END INIT INFO

   # Author: Max Lv <max.c.lv@gmail.com>

   # PATH should only include /usr/ if it runs after the mountnfs.sh script
   PATH=/sbin:/usr/sbin:/bin:/usr/bin
   DESC=shadowsocks-libev       # Introduce a short description here
   NAME=shadowsocks-libev       # Introduce the short server's name here
   DAEMON=/usr/bin/ss-server    # Introduce the server's location here
   DAEMON_ARGS=""               # Arguments to run the daemon with
   PIDFILE=/var/run/$NAME/$NAME.pid
   PIDFILE_FAST_OPEN=/var/run/$NAME/$NAME-fast-open.pid
   SCRIPTNAME=/etc/init.d/$NAME

   # Exit if the package is not installed
   [ -x $DAEMON ] || exit 0

   # Read configuration variable file if it is present
   [ -r /etc/default/$NAME ] && . /etc/default/$NAME

   [ "$START" = "yes" ] || exit 0

   : ${USER:="root"}
   : ${GROUP:="root"}

   # Load the VERBOSE setting and other rcS variables
   . /lib/init/vars.sh

   # Define LSB log_* functions.
   # Depend on lsb-base (>= 3.0-6) to ensure that this file is present.
   . /lib/lsb/init-functions

   #
   # Function that starts the daemon/service
   #
   do_start()
   {
       # Modify the file descriptor limit
       ulimit -n ${MAXFD}

       # Take care of pidfile permissions
       mkdir /var/run/$NAME 2>/dev/null || true
       chown "$USER:$GROUP" /var/run/$NAME

       # Return
       #   0 if daemon has been started
       #   1 if daemon was already running
       #   2 if daemon could not be started
       start-stop-daemon --start --quiet --pidfile $PIDFILE --chuid root:$GROUP --exec $DAEMON --test > /dev/null \
           || return 1
       start-stop-daemon --start --quiet --pidfile $PIDFILE --chuid root:$GROUP --exec $DAEMON -- \
           -c "$CONFFILE" -a "$USER" -u -f $PIDFILE $DAEMON_ARGS \
           || return 2
       start-stop-daemon --start --quiet --pidfile $PIDFILE_FAST_OPEN --chuid root:$GROUP --exec $DAEMON -- \
           -c "$CONFFILE_FAST_OPEN" -a "$USER" -u -f $PIDFILE_FAST_OPEN $DAEMON_ARGS \
           || (cat $PIDFILE | xargs kill -9; return 2)
   }

   #
   # Function that stops the daemon/service
   #
   do_stop()
   {
       # Return
       #   0 if daemon has been stopped
       #   1 if daemon was already stopped
       #   2 if daemon could not be stopped
       #   other if a failure occurred
       start-stop-daemon --stop --quiet --retry=KILL/5 --pidfile $PIDFILE --exec $DAEMON
       RETVAL="$?"
       [ "$RETVAL" = 2 ] && return 2
       # Wait for children to finish too if this is a daemon that forks
       # and if the daemon is only ever run from this initscript.
       # If the above conditions are not satisfied then add some other code
       # that waits for the process to drop all resources that could be
       # needed by services started subsequently.  A last resort is to
       # sleep for some time.
       start-stop-daemon --stop --quiet --oknodo --retry=KILL/5 --exec $DAEMON
       [ "$?" = 2 ] && return 2
       # Many daemons don't delete their pidfiles when they exit.
       rm -f $PIDFILE

       start-stop-daemon --stop --quiet --retry=KILL/5 --pidfile $PIDFILE_FAST_OPEN --exec $DAEMON
       RETVAL="$?"
       [ "$RETVAL" = 2 ] && return 2

       start-stop-daemon --stop --quiet --oknodo --retry=KILL/5 --exec $DAEMON
       [ "$?" = 2 ] && return 2

       rm -f $PIDFILE_FAST_OPEN

       return "$RETVAL"
   }

   case "$1" in
       start)
           [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC " "$NAME"
           do_start
           case "$?" in
               0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
           2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
       esac
       ;;
   stop)
       [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
       do_stop
       case "$?" in
           0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
       2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
   esac
   ;;
     status)
         status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
         ;;
     restart|force-reload)
         log_daemon_msg "Restarting $DESC" "$NAME"
         do_stop
         case "$?" in
             0|1)
                 do_start
                 case "$?" in
                     0) log_end_msg 0 ;;
                 1) log_end_msg 1 ;; # Old process is still running
             *) log_end_msg 1 ;; # Failed to start
         esac
         ;;
     *)
         # Failed to stop
         log_end_msg 1
         ;;
       esac
       ;;
   *)
       echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
       exit 3
       ;;
   esac

   :
   ```

   添加的内容：

   - 第65-67行，这里我们启动了第二个shadowsocks-libev实例。
   - 第94-101行，服务停止时，也停止第二个实例。

4. 重新启动shadowsocks：

   ```bash
   /etc/init.d/shadowsocks-libev restart
   ```

5. 使用`netstat -tlnp`能够看到ss-server监听了两个端口。

---

### ¶ The end
