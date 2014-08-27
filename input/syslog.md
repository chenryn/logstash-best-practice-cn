# 读取 Syslog 数据

syslog 可能是运维领域最流行的数据传输协议了。当你想从设备上收集系统日志的时候，syslog 应该会是你的第一选择。尤其是网络设备，比如思科 —— syslog 几乎是唯一可行的办法。

我们这里不解释如何配置你的 `syslog.conf`, `rsyslog.conf` 或者 `syslog-ng.conf` 来发送数据，而只讲如何把 logstash 配置成一个 syslog 服务器来接收数据。

有关 `rsyslog` 的用法，稍后的[类型项目](../dive_into/similar_projects.md)一节中，会有更详细的介绍。

## 配置示例

```
input {
  syslog {
    port => "514"
  }
}
```

## 运行结果

作为最简单的测试，我们先暂停一下本机的 `syslogd` (或 `rsyslogd` )进程，然后启动 logstash 进程（这样就不会有端口冲突问题）。现在，本机的 syslog 就会默认发送到 logstash 里了。我们可以用自带的 `logger` 命令行工具发送一条 "Hello World"信息到 syslog 里（即 logstash 里）。看到的 logstash 输出像下面这样：

```ruby
{
           "message" => "Hello World",
          "@version" => "1",
        "@timestamp" => "2014-08-08T09:01:15.911Z",
              "host" => "127.0.0.1",
          "priority" => 31,
         "timestamp" => "Aug  8 17:01:15",
         "logsource" => "raochenlindeMacBook-Air.local",
           "program" => "com.apple.metadata.mdflagwriter",
               "pid" => "381",
          "severity" => 7,
          "facility" => 3,
    "facility_label" => "system",
    "severity_label" => "Debug"
}
```

## 解释

Logstash 是用 `UDPSocket`, `TCPServer` 和 `LogStash::Filters::Grok` 来实现 `LogStash::Inputs::Syslog` 的。所以你其实可以直接用 logstash 配置实现一样的效果：

```
input {
  tcp {
    port => "8514"
  }
}
filter {
  grok {
    match => ["message", %{SYSLOGLINE} ]
  }
  syslog_pri { }
}
```

## 最佳实践

**强烈建议在使用 `LogStash::Inputs::Syslog` 的时候走 TCP 协议来传输数据！**

因为具体实现中，UDP 监听器只用了一个线程，而 TCP 监听器会在接收每个连接的时候都启动新的线程来处理后续步骤。

如果你已经在使用 UDP 监听器收集日志，用下行命令检查你的 UDP 接收队列大小：

```
# netstat -plnu | awk 'NR==1 || $4~/:514$/{print $2}'
Recv-Q
228096
```

228096 是 UDP 接收队列的默认最大大小，这时候 linux 内核开始丢弃数据包了！

### 小贴士

如果你实在没法切换到 TCP 协议，你可以自己写程序，或者使用其他基于异步 IO 框架(比如 libev )的项目。下面是一个简单的异步 IO 实现 UDP 监听数据输入 Elasticsearch 的示例：

<https://gist.github.com/chenryn/7c922ac424324ee0d695>
