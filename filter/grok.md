# Grok 正则捕获

Grok 是 Logstash 最重要的插件。你可以在 grok 里预定义好命名正则表达式，在稍后(grok参数或者其他正则表达式里)引用它。

## 正则表达式语法

运维工程师多多少少都会一点正则。你可以在 grok 里写标准的正则，像下面这样：

```
\s+(?<request_time>\d+(?:\.\d+)?)\s+
```

*小贴士：这个正则表达式写法对于 Perl 或者 Ruby 程序员应该很熟悉了，Python 程序员可能更习惯写 `(?P<name>pattern)`，没办法，适应一下吧。*

现在给我们的配置文件添加第一个过滤器区段配置。配置要添加在输入和输出区段之间(logstash 执行区段的时候并不依赖于次序，不过为了自己看得方便，还是按次序书写吧)：

```
input {stdin{}}
filter {
    grok {
        match => {
            "message" => "\s+(?<request_time>\d+(?:\.\d+)?)\s+"
        }
    }
}
output {stdout{}}
```

运行 logstash 进程然后输入 "begin 123.456 end"，你会看到类似下面这样的输出：

```
{
         "message" => "begin 123.456 end",
        "@version" => "1",
      "@timestamp" => "2014-08-09T11:55:38.186Z",
            "host" => "raochenlindeMacBook-Air.local",
    "request_time" => "123.456"
}
```

漂亮！不过数据类型好像不太满意……*request_time* 应该是数值而不是字符串。

我们已经提过稍后会学习用 `LogStash::Filters::Mutate` 来转换字段值类型，不过在 grok 里，其实有自己的魔法来实现这个功能！

## Grok 表达式语法

Grok 支持把预定义的 *grok 表达式* 写入到文件中，官方提供的预定义 grok 表达式见：<https://github.com/logstash/logstash/tree/v1.4.2/patterns>。

**注意：在新版本的logstash里面，pattern目录已经为空，最后一个commit提示core patterns将会由logstash-patterns-core gem来提供，该目录可供用户存放自定义patterns**

下面是从官方文件中摘抄的最简单但是足够说明用法的示例：

```
USERNAME [a-zA-Z0-9._-]+
USER %{USERNAME}
```

**第一行，用普通的正则表达式来定义一个 grok 表达式；第二行，通过打印赋值格式，用前面定义好的 grok 表达式来定义另一个 grok 表达式。**

grok 表达式的打印复制格式的完整语法是下面这样的：

```
%{PATTERN_NAME:capture_name:data_type}
```

*小贴士：dateype 目前只支持两个值：`int` 和 `float`。*

所以我们可以改进我们的配置成下面这样：

```
filter {
    grok {
        match => {
            "message" => "%{WORD} %{NUMBER:request_time:float} %{WORD}"
        }
    }
}
```

重新运行进程然后可以得到如下结果：

```
{
         "message" => "begin 123.456 end",
        "@version" => "1",
      "@timestamp" => "2014-08-09T12:23:36.634Z",
            "host" => "raochenlindeMacBook-Air.local",
    "request_time" => 123.456
}
```

这次 *request_time* 变成数值类型了。

## 最佳实践

实际运用中，我们需要处理各种各样的日志文件，如果你都是在配置文件里各自写一行自己的表达式，就完全不可管理了。所以，我们建议是把所有的 grok 表达式统一写入到一个地方。然后用 *filter/grok* 的 `patterns_dir` 选项来指明。

如果你把 "message" 里所有的信息都 grok 到不同的字段了，数据实质上就相当于是重复存储了两份。所以你可以用 `remove_field` 参数来删除掉 *message* 字段，或者用 `overwrite` 参数来重写默认的 *message* 字段，只保留最重要的部分。

重写参数的示例如下：

```
filter {
    grok {
        patterns_dir => "/path/to/your/own/patterns"
        match => {
            "message" => "%{SYSLOGBASE} %{DATA:message}"
        }
        overwrite => ["message"]
    }
}
```

## 小贴士

### 多行匹配

在和 *codec/multiline* 搭配使用的时候，需要注意一个问题，grok 正则和普通正则一样，默认是不支持匹配回车换行的。就像你需要 `=~ //m` 一样也需要单独指定，具体写法是在表达式开始位置加 `(?m)` 标记。如下所示：

```
match => {
    "message" => "(?m)\s+(?<request_time>\d+(?:\.\d+)?)\s+"
}
```

### 多项选择

有时候我们会碰上一个日志有多种可能格式的情况。这时候要写成单一正则就比较困难，或者全用 `|` 隔开又比较丑陋。这时候，logstash 的语法提供给我们一个有趣的解决方式。

文档中，都说明 logstash/filters/grok 插件的 `match` 参数应该接受的是一个 Hash 值。但是因为早期的 logstash 语法中 Hash 值也是用 `[]` 这种方式书写的，所以其实现在传递 Array 值给 `match` 参数也完全没问题。所以，我们这里其实可以传递多个正则来匹配同一个字段：

```
match => [
    "message", "(?<request_time>\d+(?:\.\d+)?)",
    "message", "%{SYSLOGBASE} %{DATA:message}",
    "message", "(?m)%{WORD}"
]
```

logstash 会按照这个定义次序依次尝试匹配，到匹配成功为止。虽说效果跟用 `|` 分割写个大大的正则是一样的，但是可阅读性好了很多。

**最后也是最关键的，我强烈建议每个人都要使用 [Grok Debugger](http://grokdebug.herokuapp.com) 来调试自己的 grok 表达式。**

![grokdebugger](http://www.elasticsearch.org/content/uploads/2014/10/Screen-Shot-2014-10-22-at-00.37.37.png)
