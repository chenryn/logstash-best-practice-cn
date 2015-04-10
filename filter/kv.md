# Key-Value 切分

在很多情况下，日志内容本身都是一个类似于 key-value 的格式，但是格式具体的样式却是多种多样的。logstash 提供 `filters/kv` 插件，帮助处理不同样式的 key-value 日志，变成实际的 LogStash::Event 数据。

## 配置示例

```
filter {
    ruby {
        init => "@kname = ['method','uri','verb']"
        code => "event.append(Hash[@kname.zip(event['request'].split(' '))])"
    }
    if [uri] {
        ruby {
            init => "@kname = ['url_path','url_args']"
            code => "event.append(Hash[@kname.zip(event['uri'].split('?'))])"
        }
        kv {
            prefix => "url_"
            source => "url_args"
            field_split => "&"
            remove_field => [ "url_args", "uri", "request" ]
        }
    }
}
```

## 解释

Nginx 访问日志中的 `$request`，通过这段配置，可以详细切分成 `method`, `url_path`, `verb`, `url_a`, `url_b` ...


