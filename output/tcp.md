# 发送网络数据(TCP)

虽然之前我们已经提到过不建议直接使用 LogStash::Inputs::TCP 和 LogStash::Outputs::TCP 做转发工作，不过在实际交流中，发现确实有不少朋友觉得这种简单配置足够使用，因而不愿意多加一层消息队列的。所以，还是把 Logstash 如何直接发送 TCP 数据也稍微提点一下。

## 配置示例

```
output {
    tcp {
        host  => "192.168.0.2"
        port  => 8888
        codec => json_lines
    }
}
```

## 配置说明

在收集端采用 tcp 方式发送给远端的 tcp 端口。这里需要注意的是，默认的 codec 选项是 **json**。而远端的 LogStash::Inputs::TCP 的默认 codec 选项却是 **plain** ！所以不指定各自的 codec ，对接肯定是失败的。

另外，由于IO BUFFER 的原因，即使是两端共同约定为 **json** 依然无法正常运行，接收端会认为一行数据没结束，一直等待直至自己 OutOfMemory ！

所以，正确的做法是，发送端指定 codec 为 **json_lines** ，这样每条数据后面会加上一个回车，接收端指定 codec 为 **json_lines** 或者 **json** 均可，这样才能正常处理。包括在收集端已经切割好的字段，也可以直接带入收集端使用了。
