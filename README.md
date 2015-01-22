Logstash 最佳实践
=======================

**Logstash is a tool for managing events and logs. You can use it to collect logs, parse them, and store them for later use (like, for searching).**
                                  -- <http://logstash.net>

虽然 Logstash 通常都是和 Kibana 以及 Elasticsearch 一起使用，其实还有很多其他的用法值得我们关注的。即便只用 ELK stack，也有很多可优化的工作需要做。

这就是本书的由来。

推荐阅读
=============

* [Elasticsearch 权威指南](http://fuxiaopang.gitbooks.io/learnelasticsearch/)
* [精通 Elasticsearch](http://shgy.gitbooks.io/mastering-elasticsearch/)
* [Kibana 中文指南](http://kibana.logstash.es/)
* [The Logstash Book](http://www.logstashbook.com/)

说明
=============

因为gitbook不让修改价钱，也不让删除。所以我把markdown格式的内容发布在github上了，见：<https://github.com/chenryn/logstash-best-practice-cn>。没法支付美元的读者可以直接下载github上的内容，然后随机打赏点支付宝：

*欢迎捐赠，作者支付宝账号：<rao.chenlin@gmail.com>*

征集
=============

logstash 社区目前比较常见的用法，有些是本人并不熟悉的，强烈希望有人可以帮助完成这部分章节：

* <del>windows 平台如何使用 nxlog 传输数据给 logstash</del>
* 如何使用 logstash-forwarder
* 如何使用 kafka 消息队列

致谢
=============

* 感谢 crazw 完成 inputs/collectd 插件介绍章节。
* 感谢 松涛 完成 ecosystem/nxlog 场景介绍章节。
* 感谢 LeiTu 完成 ecosystem/logstash-forwarder 介绍章节。
* 感谢 jingbli 完成 contrib_plugins/kafka 插件介绍章节。
