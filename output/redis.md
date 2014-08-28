# 输出到 Redis

## 配置示例

```
input { stdin {} }
output {
    redis {
        data_type => "channel"
        key => "logstash-chan-%{+yyyy.MM.dd}"
    }
}
```

## Usage

我们还是继续先用 `redis-cli` 命令行来演示 *outputs/redis* 插件的实质。

### basical use case

运行 logstash 进程，然后另一个终端启动 redis-cli 命令。输入订阅指定频道的 Redis 命令 ("SUBSCRIBE logstash-chan-2014.08.08") 后，首先会看到一个订阅成功的返回信息。如下所示：

```
# redis-cli
127.0.0.1:6379> SUBSCRIBE logstash-chan-2014.08.08
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "logstash-chan-2014.08.08"
3) (integer) 1
```

好，在运行 logstash 的终端里输入 "hello world" 字符串。切换回 redis-cli 的终端，你发现已经自动输出了一条信息：

```
1) "message"
2) "logstash-chan-2014.08.08"
3) "{\"message\":\"hello world\",\"@version\":\"1\",\"@timestamp\":\"2014-08-08T16:34:21.865Z\",\"host\":\"raochenlindeMacBook-Air.local\"}"
```

看起来是不是非常眼熟？这一串字符其实就是我们在 inputs/redis 一节中使用的那段数据。

看，这样就把 *outputs/redis* 和 *inputs/redis* 串联起来了吧！

事实上，这就是我们使用 redis 服务器作为 logstassh 架构中 broker 角色的原理。

让我们把这两节中不同配置的 logstash 进程分别在两个终端运行起来，这次不再要运行 redis-cli 命令了。在配有 *outputs/redis* 这端输入 "hello world"，配有 "inputs/redis" 的终端上，就自动输出数据了！

### notification use case

我们还可以用其他程序来订阅 redis 频道，程序里就可以随意写其他逻辑了。你可以看看 [output/juggernaut](http://logstash.net/docs/1.4.2/outputs/juggernaut) 插件的原理。这个 Juggernaut 就是基于 redis 服务器和 socket.io 框架构建的。利用它，logstash 可以直接向 webkit 等支持 socket.io 的浏览器推送告警信息。

## 扩展方式

和 `LogStash::Inputs::Redis` 一样，这里也有设置成 **list** 的方式。使用 `RPUSH` 命令发送给 redis 服务器，效果和之前展示的完全一致。包括可以调整的参数 `batch_event`，也在之前章节中讲过。这里不再重复举例。

