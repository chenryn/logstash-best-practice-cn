# Key-Value 切分



## 配置示例

```
filter {
    kv {
        target => "code"
        source => "status"
        field_split => "&"
        value_split => "="
    }
}
```
