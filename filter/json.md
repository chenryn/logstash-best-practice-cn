# JSON 编解码

在上一章，已经讲过在 codec 中使用 JSON 编码。但是，有些日志可能是一种复合的数据结构，其中只是一部分记录是 JSON 格式的。这时候，我们依然需要在 filter 阶段，单独启用 JSON 解码插件。

## 配置示例

```
filter {
    json {
        source => "message"
        target => "jsoncontent"
    }
}
```

## 运行结果

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "{\"uid\":3081609001,\"type\":\"signal\"}",
    "jsoncontent": {
        "uid": 3081609001,
        "type": "signal"
    }
}
```

## 小贴士

如果不打算使用多层结构的话，删掉 `target` 配置即可。新的结果如下：

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "{\"uid\":3081609001,\"type\":\"signal\"}",
    "uid": 3081609001,
    "type": "signal"
}
```
