# Similar Projects

## nxlog

## Graylog2

## Rsyslog

Rsyslog 是 RHEL6 开始的默认系统 syslog 应用软件(当然，RHEL 自带的版本较低，实际官方稳定版本已经到 v8 了)。官网地址：<http://www.rsyslog.com>

目前 Rsyslog 本身也支持多种输入输出方式，内部逻辑判断和模板处理。

![](http://www.rsyslog.com/common/images/rsyslog-features-imagemap.png)

不同模块插件在 rsyslog 流程中发挥作用的原理，可以阅读：<http://www.rsyslog.com/doc/master/configuration/modules/workflow.html>

流程中可以使用 [mmnormalize](http://www.rsyslog.com/doc/master/configuration/modules/mmnormalize.html) 组件来完成数据的切分(相当于 logstash 的 *filters/grok* 功能)。

使用示例见:<http://puppetlabs.com/blog/use-rsyslog-and-elasticsearch-powerful-log-aggregation>

而 normalize 语法说明见:<http://www.liblognorm.com/files/manual/index.html?sampledatabase.htm>

类似的还有 [mmjsonparse]() 组件(相当于 logstash 的 *codec/json* 功能)。

使用示例见:<http://blog.sematext.com/2013/05/28/structured-logging-with-rsyslog-and-elasticsearch/>

虽然 Rsyslog 本身可以直接输出数据给 elasticsearch，我们这里依然不推荐这种方式。因为normalize 语法还是比较简单，只支持时间，字符串，数字，ip 地址等几种。在复杂条件下远比不上完整的正则引擎。

那么，怎么使用 rsyslog 作为日志收集和传输组件，来配合 logstash 工作呢？

经笔者测试，logstash 本身的 *inputs/syslog*，*inputs/tcp* 和 *inputs/udp* 各组件在处理性能方面都有较大瓶颈，反而会拖累 rsyslog的转发性能。(如果走 udp 方式，rsyslog 端虽然可以看到 40k/s 的消费速度，实际却大多在 logstash 上被丢弃了)

不过我们可以运行多个 logstash 在本地多个端口，然后让 rsyslog 做轮训转发：

```
Ruleset( name="forwardRuleSet" ) {
    Action ( type="mmsequence" mode="instance" from="0" to="4" var="$.seq" )
    if $.seq == "0" then {
        action  (type="omfwd" Target="127.0.0.1" Port="5140" Protocol="tcp" queue.size="150000" queue.dequeuebatchsize="2000" )
    }
    if $.seq == "1" then {
        action  (type="omfwd" Target="127.0.0.1" Port="5141" Protocol="tcp" queue.size="150000" queue.dequeuebatchsize="2000" )
    }
    if $.seq == "2" then {
        action  (type="omfwd" Target="127.0.0.1" Port="5142" Protocol="tcp" queue.size="150000" queue.dequeuebatchsize="2000" )
    }
    if $.seq == "3" then {
        action  (type="omfwd" Target="127.0.0.1" Port="5143" Protocol="tcp" queue.size="150000" queue.dequeuebatchsize="2000" )
    }
}
```

经测试，这种方式可以大大提高服务器利用率，加快数据处理。

## Fluent


## Message::Passing

[Message::Passing](https://metacpan.org/pod/Message::Passing) 是一个仿造 Logstash 写的 Perl5 项目。项目早期甚至就直接照原样也叫 "Logstash" 的名字。

但实际上，Message::Passing 内部原理设计还是有所偏差的。这个项目整个基于 AnyEvent 事件驱动开发框架(即著名的 libev 库)完成，也要求所有插件不要采取阻塞式编程。所以，虽然项目开发不太活跃，插件接口不甚完善，但是性能方面却非常棒。这也是我从多个 Perl 日志处理框架中选择介绍这个的原因。

Message::Passing 有比较全的 input 和 output 插件，这意味着它可以通过多种协议跟 logstash 混跑，不过 filter 插件比较缺乏。对等于 grok 的插件叫 `Message::Passing::Filter::Regexp`( 我写的，嘿嘿)。下面是一个完整的配置示例：

```perl
use Message::Passing::DSL;
run_message_server message_chain {
    output stdout => (
        class => 'STDOUT',
    );
    output elasticsearch => (
        class => 'ElasticSearch',
        elasticsearch_servers => ['127.0.0.1:9200'],
    );
    encoder("encoder",
        class => 'JSON',
        output_to => 'stdout',
        output_to => 'es',
    );
    filter regexp => (
        class => 'Regexp',
        format => ':nginxaccesslog',
        capture => [qw( ts status remotehost url oh responsetime upstreamtime bytes )]
        output_to => 'encoder',
    );
    filter logstash => (
        class => 'ToLogstash',
        output_to => 'regexp',
    );
    decoder decoder => (
        class => 'JSON',
        output_to => 'logstash',
    );
    input file => (
        class => 'FileTail',
        output_to => 'decoder',
    );
};
```

## Heka


