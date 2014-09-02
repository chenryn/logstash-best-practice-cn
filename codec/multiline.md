# 合并多行数据(Multiline)

有些时候，应用程序调试日志会包含非常丰富的内容，为一个事件打印出很多行内容。这种日志通常都很难通过命令行解析的方式做分析。

而 logstash 正为此准备好了 *codec/multiline* 插件！

*小贴士：multiline 插件也可以用于其他类似的堆栈式信息，比如 linux 的内核日志。*

## 配置示例

```
input {
    stdin {
        codec => multiline {
            pattern => "^\["
            negate => true
            what => "previous"
        }
    }
}
```

## 运行结果

运行 logstash 进程，然后在等待输入的终端中输入如下几行数据：

```
[Aug/08/08 14:54:03] hello world
[Aug/08/09 14:54:04] hello logstash
    hello best practice
    hello raochenlin
[Aug/08/10 14:54:05] the end
```

你会发现 logstash 输出下面这样的返回：

```ruby
{
    "@timestamp" => "2014-08-09T13:32:03.368Z",
       "message" => "[Aug/08/08 14:54:03] hello world\n",
      "@version" => "1",
          "host" => "raochenlindeMacBook-Air.local"
}
{
    "@timestamp" => "2014-08-09T13:32:24.359Z",
       "message" => "[Aug/08/09 14:54:04] hello logstash\n\n    hello best practice\n\n    hello raochenlin\n",
      "@version" => "1",
          "tags" => [
        [0] "multiline"
    ],
          "host" => "raochenlindeMacBook-Air.local"
}
```

你看，后面这个事件，在 "message" 字段里存储了三行数据！

*小贴士：你可能注意到输出的事件中都没有最后的"the end"字符串。这是因为你最后输入的回车符 `\n` 并不匹配设定的 `^\[` 正则表达式，logstash 还得等下一行数据直到匹配成功后才会输出这个事件。*

## 解释

其实这个插件的原理很简单，就是把当前行的数据添加到前面一行后面，，直到新进的当前行匹配 `^\[` 正则为止。

这个正则还可以用 grok 表达式，稍后你就会学习这方面的内容。


## Log4J 的另一种方案

说到应用程序日志，log4j 肯定是第一个被大家想到的。使用 `codec/multiline` 也确实是一个办法。

不过，如果你本事就是开发人员，或者可以推动程序修改变更的话，logstash 还提供了另一种处理 log4j 的方式：[input/log4j](http://logstash.net/docs/1.4.2/inputs/log4j)。与 `codec/multiline` 不同，这个插件是直接调用了 `org.apache.log4j.spi.LoggingEvent` 处理 TCP 端口接收的数据。

## 推荐阅读

<https://github.com/elasticsearch/logstash/blob/master/patterns/java>
