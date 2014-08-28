# 读取 Redis 数据

Redis 服务器是 logstash 官方推荐的 broker 选择。Broker 角色也就意味着会同时存在输入和输出俩个插件。这里我们先学习输入插件。

`LogStash::Inputs::Redis` 支持三种 *data_type*（实际上是*redis_type*），不同的数据类型会导致实际采用不同的 Redis 命令操作：

* list            => BLPOP
* channel         => SUBSCRIBE
* pattern_channel => PSUBSCRIBE

注意到了么？**这里面没有 GET 命令！**

Redis 服务器通常都是用作 NoSQL 数据库，不过 logstash 只是用来做消息队列。所以不要担心 logstash 里的 Redis 会撑爆你的内存和磁盘。

## 配置示例

```
input {
    redis {
        data_type => "pattern_channel"
        key => "logstash-*"
        host => "192.168.0.2"
        port => 6379
        threads => 5
    }
}
```

## 使用方式

### 基本方法

首先确认你设置的 host 服务器上已经运行了 redis-server 服务，然后打开终端运行 logstash 进程等待输入数据，然后打开另一个终端，输入 `redis-cli` 命令(先安装好 redis 软件包)，在交互式提示符后面输入`PUBLISH logstash-demochan "hello world"`：

```
# redis-cli
127.0.0.1:6379> PUBLISH logstash-demochan "hello world"
```

你会在第一个终端里看到 logstash 进程输出类似下面这样的内容：

```ruby
{
       "message" => "hello world",
      "@version" => "1",
    "@timestamp" => "2014-08-08T16:26:29.399Z"
}
```

注意：这个事件里没有 **host** 字段！（或许这算是 bug……）

### 输入 JSON 数据

如果你想通过 redis 的频道给 logstash 事件添加更多字段，直接向频道发布 JSON 字符串就可以了。 `LogStash::Inputs::Redis` 会直接把 JSON 转换成事件。

继续在第二个终端的交互式提示符下输入如下内容：

```
127.0.0.1:6379> PUBLISH logstash-chan '{"message":"hello world","@version":"1","@timestamp":"2014-08-08T16:34:21.865Z","host":"raochenlindeMacBook-Air.local","key1":"value1"}'
```

你会看到第一个终端里的 logstash 进程随即也返回新的内容，如下所示：

```ruby
{
       "message" => "hello world",
      "@version" => "1",
    "@timestamp" => "2014-08-09T00:34:21.865+08:00",
          "host" => "raochenlindeMacBook-Air.local",
          "key1" => "value1"
}
```

看，新的字段出现了！现在，你可以要求开发工程师直接向你的 redis 频道发送信息好了，一切自动搞定。

### 小贴士

这里我们建议的是使用 *pattern_channel* 作为输入插件的 *data_type* 设置值。因为实际使用中，你的 redis 频道可能有很多不同的 *keys*，一般命名成 *logstash-chan-%{type}* 这样的形式。这时候 *pattern_channel* 类型就可以帮助你一次订阅全部 logstash 相关频道！

## 扩展方式

如上段"小贴士"提到的，之前两个使用场景采用了同样的配置，即数据类型为频道发布订阅方式。这种方式在需要扩展 logstash 成多节点集群的时候，会出现一个问题：**通过频道发布的一条信息，会被所有订阅了该频道的 logstash 进程同时接收到，然后输出重复内容！**

*你可以尝试再做一次上面的实验，这次在两个终端同时启动 `logstash -f redis-input.conf` 进程，结果会是两个终端都输出消息。*

这种时候，就需要用 *list* 类型。在这种类型下，数据输入到 redis 服务器上暂存，logstash 则连上 redis 服务器取走 (`BLPOP` 命令，所以只要 logstash 不堵塞，redis 服务器上也不会有数据堆积占用空间)数据。

### 配置示例

```
input {
    redis {
        batch_count => 1
        data_type => "list"
        key => "logstash-list"
        host => "192.168.0.2"
        port => 6379
        threads => 5
    }
}
```

### 使用方式

这次我们同时在两个终端运行 `logstash -f redis-input-list.conf` 进程。然后在第三个终端里启动 redis-cli 命令交互：

```
$ redis-cli 
127.0.0.1:6379> RPUSH logstash-list "hello world"
(integer) 1
```

这时候你可以看到，只有一个终端输出了结果。

连续 `RPUSH` 几次，可以看到两个终端近乎各自输出一半条目。

### 小贴士

RPUSH 支持 batch 方式，修改 logstash 配置中的 `batch_count` 值，作为示例这里只改到 2，实际运用中可以更大(事实上 `LogStash::Outputs::Redis` 对应这点的 `batch_event` 配置默认值就是 50)。

重启 logstash 进程后，redis-cli 命令中改成如下发送：

```
127.0.0.1:6379> RPUSH logstash-list "hello world" "hello world" "hello world" "hello world" "hello world" "hello world"
(integer) 3
```

可以看到，两个终端也各自输出一部分结果。而你只用了一次 RPUSH 命令。

## 推荐阅读

* <http://redis.io>
