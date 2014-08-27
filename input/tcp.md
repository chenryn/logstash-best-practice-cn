# 读取网络数据(TCP)

未来你可能会用 Redis 服务器或者其他的消息队列系统来作为 logstash broker 的角色。不过 Logstash 其实也有自己的 TCP/UDP 插件，在临时任务的时候，也算能用，尤其是测试环境。

*小贴士：虽然 `LogStash::Inputs::TCP` 用 Ruby 的 `Socket` 和 `OpenSSL` 库实现了高级的 SSL 功能，但 Logstash 本身只能在 `SizedQueue` 中缓存 20 个事件。这就是我们建议在生产环境中换用其他消息队列的原因。*

## 配置示例

```
input {
    tcp {
        port => 8888
        mode => "server"
        ssl_enable => false
    }
}
```

## 常见场景

目前来看，`LogStash::Inputs::TCP` 最常见的用法就是配合 `nc` 命令导入旧数据。在启动 logstash 进程后，在另一个终端运行如下命令即可导入数据：

```
# nc 127.0.0.1 8888 < olddata
```

这种做法比用 `LogStash::Inputs::File` 好，因为当 nc 命令结束，我们就知道数据导入完毕了。而用 input/file 方式，logstash 进程还会一直等待新数据输入被监听的文件，不能直接看出是否任务完成了。
