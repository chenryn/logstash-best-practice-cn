# 报警到 Nagios

Logstash 中有两个 output 插件是 nagios 有关的。*outputs/nagios* 插件发送数据给本机的 `nagios.cmd` 管道命令文件，*outputs/nagios_nsca* 插件则是 调用 `send_nsca` 命令以 NSCA 协议格式把数据发送给 nagios 服务器(远端或者本地皆可)。

## Nagios.Cmd

nagios.cmd 是 nagios 服务器的核心组件。nagios 事件处理和内外交互都是通过这个管道文件来完成的。

使用 CMD 方式，需要自己保证发送的 Logstash 事件符合 nagios 事件的格式。即必须在 *filter* 阶段预先准备好 `nagios_host` 和 `nagios_service` 字段；此外，如果在 *filter* 阶段也准备好 `nagios_annotation` 和 `nagios_level` 字段，这里也会自动转换成 nagios 事件信息。

```
filter {
    if [message] =~ /err/ {
        mutate {
            add_tag => "nagios"
            rename => ["host", "nagios_host"]
            replace => ["nagios_service", "logstash_check_%{type}"]
        }
    }
}
output {
    if "nagios" in [tags] {
        nagios { }
    }
}
```

如果不打算在 *filter* 阶段提供 `nagios_level` ，那么也可以在该插件中通过参数配置。

所谓 `nagios_level`，即我们通过 nagios plugin 检查数据时的返回值。其取值范围和含义如下：

* "0"，代表 "OK"，服务正常；
* "1"，代表 "WARNNING"，服务警告，一般 nagios plugin 命令中使用 `-w` 参数设置该阈值；
* "2"，代表 "CRITICAL"，服务危急，一般 nagios plugin 命令中使用 `-c` 参数设置该阈值；
* "3"，代表 "UNKNOWN"，未知状态，一般会在 timeout 等情况下出现。

默认情况下，该插件会以 "CRITICAL" 等级发送报警给 Nagios 服务器。

nagios.cmd 文件的具体位置，可以使用 `command_file` 参数设置。默认位置是 "/var/lib/nagios3/rw/nagios.cmd"。

关于和 nagios.cmd 交互的具体协议说明，有兴趣的读者请阅读 [Using external commands in Nagios](http://archive09.linux.com/feature/153285) 一文，这是《Learning Nagios 3.0》书中内容节选。

## NSCA

NSCA 是一种标准的 nagios 分布式扩展协议。分布在各机器上的 `send_nsca` 进程主动将监控数据推送给远端 nagios 服务器的 NSCA 进程。

当 Logstash 跟 nagios 服务器没有在同一个主机上运行的时候，就只能通过 NSCA 方式来发送报警了 —— 当然也必须在 Logstash 服务器上安装 `send_nsca` 命令。

nagios 事件所需要的几个属性在上一段中已经有过描述。不过在使用这个插件的时候，不要求提前准备好，而是可以在该插件内部定义参数：

```
output {
    nagios_nsca {
        nagios_host => "%{host}"
        nagios_service => "logstash_check_%{type}"
        nagios_status => "2"
        message_format => "%{@timestamp}: %{message}"
        host => "nagiosserver.domain.com"
    }
}
```

这里请注意，`host` 和 `nagios_host` 两个参数，分别是用来设置 nagios 服务器的地址，和报警信息中有问题的服务器地址。

关于 NSCA 原理，架构和配置说明，还不了解的读者请阅读官方网站 [Using NSClient++ from nagios with NSCA](http://nsclient.org/nscp/wiki/doc/usage/nagios/nsca) 一节。

## 推荐阅读

除了 nagios 以外，logstash 同样可以发送信息给其他常见监控系统。方式和 nagios 大同小异：

* *outputs/ganglia* 插件通过 UDP 协议，发送 gmetric 型数据给本机/远端的 `gmond` 或者 `gmetad`
* *outputs/zabbix* 插件调用本机的 `zabbix_sender` 命令发送
