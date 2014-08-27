# 介绍

## 历史沿革

Logstash 项目诞生于 2009 年 8 月 2 日。其作者是世界著名的运维工程师乔丹西塞(JordanSissel)，乔丹西塞当时是著名虚拟主机托管商 DreamHost 的员工，还发布过非常棒的软件打包工具 fpm，并主办着一年一度的 sysadmin advent calendar(advent calendar 文化源自基督教氛围浓厚的 Perl 社区，在每年圣诞来临的 12 月举办，从 12 月 1 日起至 12 月 24 日止，每天发布一篇小短文介绍主题相关技术)。

*小贴士：Logstash 动手很早，对比一下，scribed 诞生于 2008 年，flume 诞生于 2010 年，Graylog2 诞生于 2010 年，Fluentd 诞生于 2011 年。*

scribed 在 2011 年进入半死不活的状态，大大激发了其他各种开源日志收集处理框架的蓬勃发展，Logstash 也从 2011 年开始进入 commit 密集期并延续至今。

作为一个系出名门的产品，Logstash 的身影多次出现在 Sysadmin Weekly 上，它和它的小伙伴们 Elasticsearch、Kibana 直接成为了和商业产品 Splunk 做比较的开源项目(乔丹西塞曾经在博客上承认设计想法来自 AWS 平台上最大的第三方日志服务商 Loggy，而 Loggy 两位创始人都曾是 Splunk 员工)。

2013 年，Logstash 被 Elasticsearch 公司收购，ELK stack 正式成为官方用语(虽然还没正式命名)。Elasticsearch 本身 也是近两年最受关注的大数据项目之一，三次融资已经超过一亿美元。在 Elasticsearch 开发人员的共同努力下，Logstash 的发布机制，插件架构也愈发科学和合理。

### 小贴士

* elasticsearch 项目开始于 2010 年，其实比 logstash 还晚；
* 目前我们看到的 angularjs 版本 kibana 其实原名叫 elasticsearch-dashboard，kibana 原先是 RoR 框架的另一个项目，但作者是同一个人，换句话说，kibana 比 logstash 还早就进了 elasticsearch 名下。

## 社区文化

日志收集处理框架这么多，像 scribe 是 facebook 出品，flume 是 apache 基金会项目，都算声名赫赫。但 logstash 因乔丹西塞的个人性格，形成了一套独特的社区文化。每一个在 google groups 的 logstash-users 组里问答的人都会看到这么一句话：

**Remember: if a new user has a bad time, it's a bug in logstash.**

所以，logstash 是一个开放的，极其互助和友好的大家庭。有任何问题，尽管在 github issue，Google groups，Freenode#logstash channel 上发问就好！
