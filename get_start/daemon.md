# 长期运行

完成上一节的初次运行后，你肯定会发现一点：一旦你按下 Ctrl+C，停下标准输入输出，logstash 进程也就随之停止了。作为一个肯定要长期运行的程序，应该怎么处理呢？

*本章节问题对于一个运维来说应该属于基础知识，鉴于 ELK 用户很多其实不是运维，添加这段内容。*

办法有很多种，下面介绍四种最常用的办法：

## 标准的 service 方式

采用 RPM、DEB 发行包安装的读者，推荐采用这种方式。发行包内，都自带有 sysV 或者 systemd 风格的启动程序/配置，你只需要直接使用即可。

以 RPM 为例，`/etc/init.d/logstash` 脚本中，会加载 `/etc/init.d/functions` 库文件，利用其中的 `daemon` 函数，将 logstash 进程作为后台程序运行。

所以，你只需把自己写好的配置文件，统一放在 `/etc/logstash/` 目录下(注意目录下所有配置文件都应该是 **.conf** 结尾，且不能有其他文本文件存在。因为 logstash agent 启动的时候是读取全文件夹的)，然后运行 `service logstash start` 命令即可。

## 最基础的 nohup 方式

这是最简单的方式，也是 linux 新手们很容易搞混淆的一个经典问题：

```
command
command > /dev/null
command > /dev/null 2>&1
command &
command > /dev/null &
command > /dev/null 2>&1 &
command &> /dev/null
nohup command &> /dev/null
```

请回答以上命令的异同……

具体不一一解释了。直接说答案，想要维持一个长期后台运行的 logstash，你需要同时在命令前面加 `nohup`，后面加 `&`。

## 更优雅的 SCREEN 方式

screen 算是 linux 运维一个中高级技巧。通过 screen 命令创建的环境下运行的终端命令，其父进程不是 sshd 登录会话，而是 screen 。这样就可以即避免用户退出进程消失的问题，又随时能重新接管回终端继续操作。

创建独立的 screen 命令如下：

```
screen -dmS elkscreen_1
```

接管连入创建的 `elkscreen_1` 命令如下：

```
screen -r elkscreen_1
```

然后你可以看到一个一模一样的终端，运行 logstash 之后，不要按 Ctrl+C，而是按 Ctrl+A+D 键，断开环境。想重新接管，依然 `screen -r elkscreen_1` 即可。

如果创建了多个 screen，查看列表命令如下：

```
screen -list
```

## 最推荐的 daemontools 方式

不管是 nohup 还是 screen，都不是可以很方便管理的方式，在运维管理一个 ELK 集群的时候，必须寻找一种尽可能简洁的办法。所以，对于需要长期后台运行的大量程序(注意大量，如果就一个进程，还是学习一下怎么写 init 脚本吧)，推荐大家使用一款 daemontools 工具。

daemontools 是一个软件名称，不过配置略复杂。所以这里我其实是用其名称来指代整个同类产品，包括但不限于 python 实现的 supervisord，perl 实现的 ubic，ruby 实现的 god 等。

以 supervisord 为例，因为这个出来的比较早，可以直接通过 EPEL 仓库安装。

```
yum -y install supervisord --enablerepo=epel
```

在 `/etc/supervisord.conf` 配置文件里添加内容，定义你要启动的程序：

```
[program:elkpro_1]
environment=LS_HEAP_SIZE=5000m
directory=/opt/logstash
command=/opt/logstash/bin/logstash -f /etc/logstash/pro1.conf --pluginpath /opt/logstash/plugins/ -w 10 -l /var/log/logstash/pro1.log
[program:elkpro_2]
environment=LS_HEAP_SIZE=5000m
directory=/opt/logstash
command=/opt/logstash/bin/logstash -f /etc/logstash/pro2.conf --pluginpath /opt/logstash/plugins/ -w 10 -l /var/log/logstash/pro2.log
```

然后启动 `service supervisord start` 即可。

logstash 会以 supervisord 子进程的身份运行，你还可以使用 `supervisorctl` 命令，单独控制一系列 logstash 子进程中某一个进程的启停操作：

`supervisorctl stop elkpro_2`

