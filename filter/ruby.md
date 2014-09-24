# 随心所欲的 Ruby 处理

如果你稍微懂那么一点点 Ruby 语法的话，*filters/ruby* 插件将会是一个非常有用的工具。

比如你需要稍微修改一下 `LogStash::Event` 对象，但是又不打算为此写一个完整的插件，用 *filters/ruby* 插件绝对感觉良好。

## 配置示例

```
filter {
    ruby {
        init => "@kname = ['client','servername','url','status','time','size','upstream','upstreamstatus','upstreamtime','referer','xff','useragent']"
        code => "event.append(Hash[@kname.zip(event['message'].split('|'))])"
    }
}
```

官网示例是一个比较有趣但是没啥大用的做法 —— 随机取消 90% 的事件。

所以上面我们给出了一个有用而且强大的实例。

## 解释

通常我们都是用 *filters/grok* 插件来捕获字段的，但是正则耗费大量的 CPU 资源，很容易成为 Logstash 进程的瓶颈。

而实际上，很多流经 Logstash 的数据都是有自己预定义的特殊分隔符的，我们可以很简单的直接切割成多个字段。

*filters/mutate* 插件里的 "split" 选项只能切成数组，后续很不方便使用和识别。而在 *filters/ruby* 里，我们可以通过 "init" 参数预定义好由每个新字段的名字组成的数组，然后在 "code" 参数指定的 Ruby 语句里通过两个数组的 zip 操作生成一个哈希并添加进数组里。短短一行 Ruby 代码，可以减少 50% 以上的 CPU 使用率。

*filters/ruby* 插件用途远不止这一点，下一节你还会继续见到它的身影。

## 更多实例

*2014 年 09 年 23 日新增*

```
filter{
    date {
        match => ["datetime" , "UNIX"]
    }
    ruby {
        code => "event.cancel if 5 * 24 * 3600 < (event['@timestamp']-::Time.now).abs"
    }
}
```

在实际运用中，我们几乎肯定会碰到出乎意料的输入数据。这都有可能导致 Elasticsearch 集群出现问题。

当数据格式发生变化，比如 UNIX 时间格式变成 UNIX_MS 时间格式，会导致 logstash 疯狂创建新索引，集群崩溃。

或者误输入过老的数据时，因为一般我们会 close 几天之前的索引以节省内存，必要时再打开。而直接尝试把数据写入被关闭的索引会导致内存问题。

这时候我们就需要提前校验数据的合法性。上面配置，就是用于过滤掉时间范围与当前时间差距太大的非法数据的。
