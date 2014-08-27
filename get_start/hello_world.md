# Hello World

和绝大多数 IT 技术介绍一样，我们以一个输出 "hello world" 的形式开始我们的 logstash 学习。

## 运行

在终端中，像下面这样运行命令来启动 Logstash 进程：

```
# bin/logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

然后你会发现终端在等待你的输入。没问题，敲入 **Hello World**，回车，然后看看会返回什么结果！

## 结果

```ruby
{
       "message" => "Hello World",
      "@version" => "1",
    "@timestamp" => "2014-08-07T10:30:59.937Z",
          "host" => "raochenlindeMacBook-Air.local",
}
```

没错！你搞定了！这就是全部你要做的。

## 解释

每位系统管理员都肯定写过很多类似这样的命令：`cat randdata | awk '{print $2}' | sort | uniq -c | tee sortdata`。这个管道符 `|` 可以算是 Linux 世界最伟大的发明之一(另一个是“一切皆文件”)。

Logstash 就像管道符一样！

你**输入**(就像命令行的 `cat` )数据，然后处理**过滤**(就像 `awk` 或者 `uniq` 之类)数据，最后**输出**(就像 `tee` )到其他地方。

*当然实际上，Logstash 是用不同的线程来实现这些的。如果你运行 `top` 命令然后按下 `H` 键，你就可以看到下面这样的输出：*

```
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                          
21401 root      16   0 1249m 303m  10m S 18.6  0.2 866:25.46 |worker                           
21467 root      15   0 1249m 303m  10m S  3.7  0.2 129:25.59 >elasticsearch.                   
21468 root      15   0 1249m 303m  10m S  3.7  0.2 128:53.39 >elasticsearch.                   
21400 root      15   0 1249m 303m  10m S  2.7  0.2 108:35.80 <file                             
21403 root      15   0 1249m 303m  10m S  1.3  0.2  49:31.89 >output                           
21470 root      15   0 1249m 303m  10m S  1.0  0.2  56:24.24 >elasticsearch.
```

*小贴士：logstash 很温馨的给每个线程都取了名字，输入的叫<xx，输出的叫>xx，过滤的叫|xx*

数据在线程之间以 **事件** 的形式流传。不要叫*行*，因为 logstash 可以处理[多行](../codec/multiline.md)事件。

Logstash 会给事件添加一些额外信息。最重要的就是 **@timestamp**，用来标记事件的发生时间。而 **host** 标记事件发生在哪里。

你可以随意给事件添加字段或者从事件里删除字段。事实上事件就是一个 Ruby 对象，或者更简单的理解为就是一个哈希也行。

*小贴士：每个 logstash 插件，不管是输入、过滤还是输出，都会有两个方法叫 `add_field` 和 `remove_field`。*

## 推荐阅读

* [the life of an event](http://logstash.net/docs/1.4.2/life-of-an-event)
