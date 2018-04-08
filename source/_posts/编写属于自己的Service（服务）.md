---
title: <font color=#0099ff size=6 face="微软雅黑">编写属于自己的Service(服务)</font>
date: 2017-05-31 21:10:26
categories: Linux
tags: [Linux,Service,后台服务,Chkconfig]
---

用Apache james搭建自己的邮件服务器，需要运行james解压包中/bin目录下的run.sh来开启邮件服务器的，然而只能在前台开启，开启后就会邮件服务器会一直运行，也就是说这个终端窗口就不能做别的事情了，除非另外开一个终端窗口。怎样解决呢？其实可以用nohup命令让其变为后台服务。比如说我的james放在/usr/local目录下，那么我可以这么做：
```shell
nohup sh /usr/local/james/bin/run.sh > RUNNING_REPORT 2>&1 &
```
nohup可以让某个程序在后台运行，用&在程序结尾让程序自动运行。中间的> RUNNING_REPORT是将原程序的标准输出重定向输出到RUNNING_REPORT文件中去，2>&1是指把标准错误输出也重定向到标准输出中去，也就是说如果有错误信息，最终也将输出到RUNNING_REPORT文件中。

像邮件服务器这样一个持续运行的服务更像是一个系统服务，所以我们可以把它写成自己的service。使用到的是chkconfig，chkconfig命令可以用来检查、设置系统的各种服务。一个service有启动、停止，查询状态和重启这些基础命令，所以这里有一种通用的写法：
```shell
# chkconfig: 2345 90 10
# description:auto_run

#因为james需要java环境，如果找不到就重定向到一个已知的java环境
export JAVA_HOME=${JAVA_HOME:-/usr/java/latest}
export JS_HOME=/usr/james
export JS_BIN_DIR=$JS_HOME/bin
#RUNNING_REPORT用来记录程序的标准输出，起到log的作用
export JS_PID_FILE=$JS_HOME/RUNNING_REPORT

#启动
function start {
    if [ -f "$JS_PID_FILE" ]; then
        echo "James PID file already exists. Skipped start process"
        exit 0
    fi

    echo "Starting james service..."
    cd $JS_BIN_DIR
    nohup sh run.sh > $JS_PID_FILE 2>&1 &
    sleep 1
}

#停止
function stop {
    if [ -f $JS_PID_FILE ]; then
        set +e
        kill -9 `cat $JS_PID_FILE` > /dev/null 2>&1 &
        rm -f $JS_PID_FILE
        set -e
        sleep 1
    fi
}

#状态
function status {
    if [ -f $JS_PID_FILE ]; then
        echo "James is running"
    else
        echo "James is not running"
    fi
}

#重启
function restart {
    stop
    start
}

case "$1" in
    start)
    start
    ;;
    stop)
    stop
    ;;
    status)
    status
    ;;
    restart)
    restart
    ;;
    *)
    echo "Usage: $0 start|stop|status|restart"
esac
```
这个脚本应该很清晰，文件只需要要无后缀保存就可以了，比如保存为mailServer，然后把它放到/etc/init.d目录下，这样就可以用下面的命令来启动了：
```shell
service NAME start
```
其中的NAME就是文件名。

下面来详细讲一下chkconfig命令：
严格来说chkconfig命令主要用来更新（启动或停止）和查询系统服务的运行级信息。首先要弄明白什么是运行级，在Linux中，运行级别就是操作系统当前正在运行的功能级别。级别可以从0到6，具有不同的功能。这些级别定义在/ect/inittab文件中。这个文件是init程序寻找的主要文件，而最先运行的服务是那些放在/ect/rc.d目录下的文件。

**Linux下的7个运行级别：**
**0**：系统停机状态，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
**1**：单用户工作状态，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。
**2**：无网络连接的多用户命令行模式
**3**：有网络连接的多用户命令行模式
**4**：系统未使用状态，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
**5**：带图形界面的多用户模式。
**6**：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动。运行init 6机器就会重启。

在目录/etc/init.d下有许多服务器脚本程序，一般称为服务(service)
在/etc/下有7个名为rc[N].d(N为0~6)的目录，对应系统的7个运行级别
rc[N].d目录下都是一些符号链接文件，这些链接文件都指向init.d目录下的service脚本文件，命名规则为K+nn+服务名或S+nn+服务名，其中nn为两位数字。
系统会根据指定的运行级别进入对应的rc[N].d目录，并按照文件名顺序检索目录下的链接文件：对于以K开头的文件，系统将终止对应的服务； 对于以S开头的文件，系统将启动对应的服务

上面代码中第一行中的2345就是指运行级别。而之后的90代表Start的优先级，10代表Kill（Stop）的优先级。如果启动优先级配置的太小可能会启动不成功，因为可能它依赖的一些环境（比如网络环境）还没启动，从而导致启动失败。

Chkconfig有五个很明确的功能：为管理增加一个新的功能、删除一个功能、列出当前服务的启动信息、改变一个服务的启动信息和检测特殊服务的启动状态。
对应的命令行;
```shell
chkconfig --add name
chkconfig --del name
chkconfig --list [name]
chkconfig [--level levels] name <on|off|reset>
chkconfig [--level levels] name
```
将需要自动启动的脚本/etc/init.d目录下，然后使用如下命令
```shell
chkconfig --add filename
```
这样就可以使服务自动注册开机启动和关机关闭。实质就是在rc0.d-rc6.d目录下生成一些文件连接，这些连接连接到/etc/init.d目录下指定文件的shell脚本。

所以如果我们想把上面写过的mail服务设置成开机自启，就需要先把它放到/etc/init.d目录下，然后赋予它可执行的权限：
```shell
chmod +x mailserver
```
然后执行
```shell
chkconfig --add mailserver
```
命令，或者直接在rc0.d-rc6.d目录下分别创建文件连接:
ln -s /etc/rc.d/init.d/auto_run /etc/rc.d/rc2.d/S99mailserver
ln -s /etc/rc.d/init.d/auto_run /etc/rc.d/rc3.d/S99mailserver
ln -s /etc/rc.d/init.d/auto_run /etc/rc.d/rc5.d/S99mailserver
ln -s /etc/rc.d/init.d/auto_run /etc/rc.d/rc0.d/K01mailserver
ln -s /etc/rc.d/init.d/auto_run /etc/rc.d/rc6.d/K01mailserver
这样系统在启动的时候，就会运行auto_run 并加上start参数，等同于执行命令mailserver start。
在系统关闭的时候，就会运行auto_run，并加上stop参数，等同于运行命令mailserver stop。