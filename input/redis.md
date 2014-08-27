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
        batch_count => 1
        data_type => "pattern_channel"
        key => "logstash-*"
        host => "192.168.0.2"
        port => 6379
        threads => 5
    }
}
```

## Usage

### Basic usage

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

### transfter JSON data

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

## Tips:

这里我们建议的是使用 *pattern_channel* 作为输入插件的 *data_type* 设置值。因为实际使用中，你的 redis 频道可能有很多不同的 *keys*，一般命名成 *logstash-chan-%{type}* 这样的形式。这时候 *pattern_channel* 类型就可以帮助你一次订阅全部 logstash 相关频道！

另外，可能你也会喜欢用 *list* 类型，在这种类型下，你可以用 redis 的 `LLEN` 命令监控你 Redis 服务器中缓存的数据大小。

## 推荐阅读

* <http://redis.io>
