# Logstash Forwarder

Redis 已经帮我们解决了很多的问题，而且也很轻量，为什么我们还需要 logstash-forwarder 呢？

> Redis provides simple authentication but no transport-layer encryption or authorization. This is perfectly fine in trusted environments. However, if you're connecting to Redis between datacenters you will probably want to use encryption.

简而言之他很好，但是他不 secure。

现在看看我们如何来配置 logstash-forwarder。

## indexer 端配置

在 logstash 作为 indexer server 角色的这端，我们首先需要生成证书：

    cd /etc/pki/tls
    sudo openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt

然后把证书发送到准备运行 logstash-forwarder 的 shipper 端服务器上去：

    scp private/logstash-forwarder.key root@target_server_ip:/etc/pki/tls/private
    scp certs/logstash-forwarder.crt root@target_server_ip:/etc/pki/tls/certs

然后创建 logstash 的配置文件。监听部分 /etc/logstash/conf.d/02-lumberjack-input.conf ，内容如下：

```
input {
  lumberjack {
    port => 5000
    type => "anything"
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}
```

以上，我们在 logstash 这端已经配置完成。运行 `logstash -f /etc/logstash/conf.d/` 即可。

*小知识：lumberjack 是 logstash-forwarder 还没用 Golang 重写之前的名字*

## shipper 端配置

我们现在登入到我们需要传送 log 的机器上，我们已在之前的步骤中发送了 logstash 的 crt 过来。

### logstash-forwarder 安装

首先，我们需要安装 logstash-forwarder 软件。官方都已经提供了软件仓库可用。在 Redhat 机器上只需要添加一个 */etc/yum.repos.d/logstash-forwarder.repo*，内容如下：

```ini
[logstash-forwarder]
name=logstash-forwarder
baseurl=http://packages.elasticsearch.org/logstash-forwarder/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
```

然后运行安装命令即可：

    sudo yum install -y logstash-forwarder

你可以从我提供的 gist 中下载已经更改的 init script 或者使用 rpm 中提供的脚本 [logstash-forwader](https://gist.github.com/ae30a4c1a1f342df1274.git).

### logstash-forwarder 配置

logstash-forwarder 的配置文件是纯 JSON 格式。因为其轻量级的设计目的，所以可配置项很少。下面是一个 */etc/logstash-forwarder* 配置示例：

```json
{
  "network": {
    "servers": [ "10.18.10.2:5000" ],
      "timeout": 15,
      "ssl ca" : "/etc/pki/tls/certs/logstash-forwarder.crt"
      "ssl key": "/etc/pki/tls/private/logstash-forwarder.key"
  },
  "files": [
    {
      "paths": [
        "/var/log/message",
      "/var/log/secure"
        ],
      "fields": { "type": "syslog" }
    }
  ]
}
```

我们已完成了配置，当 `sudo service logstash-forwarder start` 之后，你就可以在 kibana 上看到你的日志了

### logstash-forwarder 配置说明

配置中，主要包括下面几个可用配置项：

* network.servers: 用来指定远端(即 logstash indexer 角色)服务器的 IP 地址和端口。这里可以写数组，但是 logstash-forwarder 只会随机选一台作为对端发送数据，一直到对端故障，才会重选其他服务器。
* network.ssl\*: 网络交互中使用的 SSL 证书路径。
* files.\*.paths: 读取的文件路径。
  logstash-forwarder 只支持两种输入，一种就是示例中用的文件方式，和 logstash 一样也支持 glob 路径，即 `"/var/log/*.log"` 这样的写法；一种是标准输入，写法为 `"paths": [ "-" ]`
* files.\*.fields: 给每条日志添加的固定字段，相当于 logstash 里的 `add_field` 参数。
  注意示例中添加的是 **type** 字段。在 logstash-forwarder 里添加的字段是优先于 LogStash::Inputs::Lumberjack 配置里定义的字段的。所以，在本例中，即便你在 indexer 上定义 type 为 "anything"。事件的实际 type 依然是这里添加的 "syslog"。这也意味着，你在 indexer 上如果做后续判断，应该是这样：

```
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    }
  }
}
```

## 安全性提示

虽然 ssl 是可信任的，但是当 hacker 得到你一台机器上的证书后，他可以畅通无阻，建议对每台机器都签发单独的证书，如果你忙的过来的话:)
