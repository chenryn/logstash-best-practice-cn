# 采用 JSON 编码

在早期的版本中，有一种降低 logstash 过滤器的 CPU 负载消耗的做法盛行于社区(在当时的 cookbook 上有专门的一节介绍)：**直接输入预定义好的 JSON 数据，这样就可以省略掉 filter/grok 配置！**

这个建议依然有效，不过在当前版本中需要稍微做一点配置变动 —— 因为现在有专门的 *codec* 设置。

## 配置示例

社区常见的示例都是用的 Apache 的 customlog。不过我觉得 Nginx 是一个比 Apache 更常用的新型 web 服务器，所以我这里会用 nginx.conf 做示例：

```
logformat json '{"@timestamp":"$time_iso8601",'
               '"@version":"1",'
               '"host":"$server_addr",'
               '"client":"$remote_addr",'
               '"size":$body_bytes_sent,'
               '"responsetime":$request_time,'
               '"domain":"$host",'
               '"url":"$uri",'
               '"status":"$status"}';
access_log /var/log/nginx/access.log_json json;
```

*注意：在 `$request_time` 和 `$body_bytes_sent` 变量两头没有双引号 `"`，这两个数据在 JSON 里应该是数值类型！*

重启 nginx 应用，然后修改你的 input/file 区段配置成下面这样：

```
input {
    file {
        path => "/var/log/nginx/access.log_json""
        codec => "json"
    }
}
```

## 运行结果

下面访问一下你 nginx 发布的 web 页面，然后你会看到 logstash 进程输出类似下面这样的内容：

```ruby
{
      "@timestamp" => "2014-03-21T18:52:25.000+08:00",
        "@version" => "1",
            "host" => "raochenlindeMacBook-Air.local",
          "client" => "123.125.74.53",
            "size" => 8096,
    "responsetime" => 0.04,
          "domain" => "www.domain.com",
             "url" => "/path/to/file.suffix",
          "status" => "200"
}
```

## 小贴士

对于一个 web 服务器的访问日志，看起来已经可以很好的工作了。不过如果 Nginx 是作为一个代理服务器运行的话，访问日志里有些变量，比如说 `$upstream_response_time`，可能不会一直是数字，它也可能是一个 `"-"` 字符串！这会直接导致 logstash 对输入数据验证报异常。

有两个办法解决这个问题：

1. 用 `sed` 在输入之前先替换 `-` 成 `0`。

运行 logstash 进程时不再读取文件而是标准输入，这样命令就成了下面这个样子：

```
tail -F /var/log/nginx/proxy_access.log_json \
    | sed 's/upstreamtime":-/upstreamtime":0/' \
    | /usr/local/logstash/bin/logstash -f /usr/local/logstash/etc/proxylog.conf
```

2. 日志格式中统一记录为字符串格式(即都带上双引号 `"`)，然后再在 logstash 中用 `filter/mutate` 插件来变更应该是数值类型的字符字段的值类型。

有关 `LogStash::Filters::Mutate` 的内容，本书稍后会有介绍。
