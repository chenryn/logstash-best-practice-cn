# 时间处理(Date)

之前章节已经提过，*filters/date* 插件可以用来转换你的日志记录中的时间字符串，变成 `LogStash::Timestamp` 对象，然后转存到 `@timestamp` 字段里。

**注意：因为在稍后的 outputs/elasticsearch 中常用的 `%{+YYYY.MM.dd}` 这种写法必须读取 `@timestamp` 数据，所以一定不要直接删掉这个字段保留自己的字段，而是应该用 filters/date 转换后删除自己的字段！**

这在导入旧数据的时候固然非常有用，而在实时数据处理的时候同样有效，因为一般情况下数据流程中我们都会有缓冲区，导致最终的实际处理时间跟事件产生时间略有偏差。

*小贴士：个人强烈建议打开 Nginx 的 access_log 配置项的 buffer 参数，对极限响应性能有极大提升！*

## 配置示例

*filters/date* 插件支持五种时间格式：

### ISO8601

类似 "2011-04-19T03:44:01.103Z" 这样的格式。具体Z后面可以有 "08:00"也可以没有，".103"这个也可以没有。常用场景里来说，Nginx 的 *log_format* 配置里就可以使用 `$time_iso8601` 变量来记录请求时间成这种格式。

### UNIX

UNIX 时间戳格式，记录的是从 1970 年起始至今的总秒数。Squid 的默认日志格式中就使用了这种格式。

### UNIX_MS

这个时间戳则是从 1970 年起始至今的总毫秒数。据我所知，JavaScript 里经常使用这个时间格式。

### TAI64N

TAI64N 格式比较少见，是这个样子的：`@4000000052f88ea32489532c`。我目前只知道常见应用中， qmail 会用这个格式。

### Joda-Time 库

Logstash 内部使用了 Java 的 Joda 时间库来作时间处理。所以我们可以使用 Joda 库所支持的时间格式来作具体定义。Joda 时间格式定义见下表：

#### 时间格式

|Symbol  |Meaning                      |Presentation  |Examples|
|--------|-----------------------------|--------------|-------|
|G       |era                          |text          |AD|
|C       |century of era (>=0)         |number        |20|
|Y       |year of era (>=0)            |year          |1996|
|x       |weekyear                     |year          |1996|
|w       |week of weekyear             |number        |27|
|e       |day of week                  |number        |2|
|E       |day of week                  |text          |Tuesday; Tue|
|y       |year                         |year          |1996|
|D       |day of year                  |number        |189|
|M       |month of year                |month         |July; Jul; 07|
|d       |day of month                 |number        |10|
|a       |halfday of day               |text          |PM|
|K       |hour of halfday (0~11)       |number        |0|
|h       |clockhour of halfday (1~12)  |number        |12|
|H       |hour of day (0~23)           |number        |0|
|k       |clockhour of day (1~24)      |number        |24|
|m       |minute of hour               |number        |30|
|s       |second of minute             |number        |55|
|S       |fraction of second           |number        |978|
|z       |time zone                    |text          |Pacific Standard Time; PST|
|Z       |time zone offset/id          |zone          |-0800; -08:00; America/Los_Angeles|
|'       |escape for text              |delimiter     ||
|''      |single quote                 |literal       |'|

<http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html>

下面我们写一个 Joda 时间格式的配置作为示例：

```
filter {
    grok {
        match => ["message", "%{HTTPDATE:logdate}"]
    }
    date {
        match => ["logdate", "dd/MMM/yyyy:HH:mm:ss Z"]
    }
}
```

**注意：时区偏移量只需要用一个字母 `Z` 即可。**

## 时区问题的解释

很多中国用户经常提一个问题：为什么 @timestamp 比我们早了 8 个小时？怎么修改成北京时间？

其实，Elasticsearch 内部，对时间类型字段，是**统一采用 UTC 时间，存成 long 长整形数据的**！对日志统一采用 UTC 时间存储，是国际安全/运维界的一个通识——欧美公司的服务器普遍广泛分布在多个时区里——不像中国，地域横跨五个时区却只用北京时间。

对于页面查看，ELK 的解决方案是在 Kibana 上，读取浏览器的当前时区，然后在页面上转换时间内容的**显示**。

所以，建议大家接受这种设定。否则，即便你用 `.getLocalTime` 修改，也还要面临在 Kibana 上反过去修改，以及 Elasticsearch 原有的 `["now-1h" TO "now"]` 这种方便的搜索语句无法正常使用的尴尬。

以上，请读者自行斟酌。
