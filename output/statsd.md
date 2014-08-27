# 输出到 Statsd

Statsd 最早是 2008 年 Flickr 公司用 Perl 写的针对 graphite、datadog 等监控数据后端存储开发的前端网络应用，2011 年 Etsy 公司用 nodejs 重构。用于接收、写入、读取和聚合时间序列数据，包括即时值和累积值等。

## 配置示例

```
output {
    statsd {
        host => "statsdserver.domain.com"
        namespace => "logstash"
        sender => "%{host}"
        increment => ["httpd.response.%{status}"]
    }
}
```

## 解释

Graphite 以树状结构存储监控数据，所以 statsd 也是如此。所以发送给 statsd 的数据的 key 也一定得是 "first.second.tree.four" 这样的形式。而在 *outputs/statsd* 插件中，就会以三个配置参数来拼接成这种形式：

```
    namespace.sender.metric
```

其中 namespace 和 sender 都是直接设置的，而 metric 又分为好几个不同的参数可以分别设置。statsd 支持的 metric 类型如下：

### metric 类型

* increment

示例语法：`increment => ["nginx.status.%{status}"]`

* decrement

语法同 increment。

* count

示例语法：`count => {"nginx.bytes" => "%{bytes}"}`

* gauge

语法同 count。

* set

语法同 count。

* timing

语法同 count。

关于这些 metric 类型的详细说明，请阅读 statsd 文档：<https://github.com/etsy/statsd/blob/master/docs/metric_types.md>。

## 推荐阅读

* Etsy 发布 nodejs 版本 statsd 的博客：[Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/)
* Flickr 发布 statsd 的博客：[Counting & Timing](http://code.flickr.net/2008/10/27/counting-timing/)
