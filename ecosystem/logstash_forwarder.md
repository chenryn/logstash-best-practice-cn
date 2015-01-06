##Logstash Forwarder

Redis已经帮我们解决了很多的问题，而且也很轻量，为什么我们还需要logstash－forwarder呢？

>Redis provides simple authentication but no transport-layer encryption or authorization. This is perfectly fine in trusted environments. However, if you're connecting to Redis between datacenters you will probably want to use encryption.

>他很好，但是他不secure


我们如何来配置logstash－forwarder。
在logstash server这端，我们首先需要生成证书。

```json
cd /etc/pki/tls; sudo openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
scp private/logstash-forwarder.key root@target_server_ip:/etc/pki/tls/private
sudo cat > /etc/logstash/conf.d/02-lumberjack-input.conf <<EOF
input {
  lumberjack {
    port => 5000
      type => "logs"
      ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
      ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}
EOF
```
添加对应的logstash的syslog type

```json
sudo cat > /etc/logstash/conf.d/03-syslog.conf <<EOF
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_ho
        stname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}
        " }
        add_field => [ "received_at", "%{@timestamp}" ]
          add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
        remove_field => ["timestamp"]
    }
    date {
      match => ["timestamp8601", "ISO8601"]
        remove_field => ["timestamp8601"]
    }
  }
}
EOF
```
*注意：在新版本的logstash里面，pattern目录已经为空，最后一个commit提示core patterns将会由logstash－patterns－core gem来提供，该目录可供用户存放自定义patterns*

```json
sudo cat > /etc/logstash/conf.d/30-lumberjack-output.conf <<EOF
output {
  elasticsearch {
    host => "localhost"
      port => "9200"
      protocol => "http"
      index => "logstash-%{+YYYY.MM.dd}-%{type}"
  }
  stdout { codec => rubydebug }
}
EOF
sudo service logstash restart
```

以上，我们在logstash这段已经配置完成。
*请在重启前一定执行/opt/logstash/bin/logstash -f /ect/logstash/conf.d -t已确保你的配置文件是正确的， 或者访问jsonlint.com在线教研你的json*

我们现在登入到我们需要传送log的机器上，我们已在之前的步骤中发送了logstash的crt


```json
sudo cat > /etc/yum.repos.d/logstash-forwarder.repo << EOF
[logstash-forwarder]
name=logstash-forwarder
baseurl=http://packages.elasticsearch.org/logstash-forwarder/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
EOF
sudo yum install -y logstash-forwarder
```
*你可以从我提供的gist中下载已经更改的init script或者使用rpm中提供的脚本[logstash-forwader](https://gist.github.com/ae30a4c1a1f342df1274.git).*


```json
sudo cat > /etc/logstash-forwarder << EOF
{
  "network": {
    "servers": [ "10.18.10.2:5000" ],
      "timeout": 15,
      "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt"
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
EOF
sudo service logstash-forwarder start
```

我们已完成了配置，当重启logstash－forwarder之后，你就可以在kibana上看到你的日志了

请记住一下

>虽然ssl是可信任的，但是当hacker得到你一台机器上的证书后，他可以畅通无阻，建议对没太机器都签发相应证书，如果你忙的过来的话:)
