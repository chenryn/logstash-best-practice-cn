# Message::Passing

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

