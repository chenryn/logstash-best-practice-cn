# Heka

heka 是 Mozilla 公司仿造 logstash 设计，用 Golang 重写的一个开源项目。同样采用了input -> decoder -> filter -> encoder -> output 的流程概念。其特点在于，在中间的 decoder/filter/encoder 部分，设计了 sandbox 概念，可以采用内嵌 lua 脚本做这一部分的工作，降低了全程使用静态 Golang 编写的难度。此外，其 filter 阶段还提供了一些监控和统计报警功能。

官网地址见：<http://hekad.readthedocs.org/>

下面是同样的处理逻辑，通过 syslog 接收 nginx 访问日志，解析并存储进 Elasticsearch，heka 配置文件如下：

```ini
[hekad]
maxprocs = 48

[TcpInput]
address = ":514"
parser_type = "token"
decoder = "shipped-nginx-decoder"

[shipped-nginx-decoder]
type = "MultiDecoder"
subs = ['RsyslogDecoder', 'nginx-access-decoder']
cascade_strategy = "all"
log_sub_errors = true

[RsyslogDecoder]
type = "SandboxDecoder"
filename = "lua_decoders/rsyslog.lua"
    [RsyslogDecoder.config]
    type = "nginx.access"
    template = '<%pri%>%TIMESTAMP% %HOSTNAME% %syslogtag%%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n'
    tz = "Asia/Shanghai"

[nginx-access-decoder]
type = "SandboxDecoder"
filename = "lua_decoders/nginx_access.lua"

    [nginx-access-decoder.config]
    type = "combined"
    user_agent_transform = true
    log_format = '[$time_local]`$http_x_up_calling_line_id`"$request"`"$http_user_agent"`$staTus`[$remote_addr]`$http_x_log_uid`"$http_referer"`$request_time`$body_bytes_sent`$http_x_forwarded_proto`$http_x_forwarded_for`$request_uid`$http_host`$http_cookie`$upstream_response_time'

[ESLogstashV0Encoder]
es_index_from_timestamp = true
fields = ["Timestamp", "Payload", "Hostname", "Fields"]
type_name = "%{Type}"

[ElasticSearchOutput]
message_matcher = "Type == 'nginx.access'"
server = "http://eshost.example.com:9200"
encoder = "ESLogstashV0Encoder"
flush_interval = 50
flush_count = 5000
```

heka 目前仿造的还是旧版本的 logstash schema 设计，所有切分字段都存储在 `@fields` 下。

经测试，其处理性能跟开启了多线程 filters 的 logstash 进程类似，都在每秒 30000 条。
