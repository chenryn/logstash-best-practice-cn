# 为什么用 JRuby？能用 MRI 运行么？

对日志处理框架有一些了解的都知道，大多数框架都是用 Java 写的，毕竟做大规模系统 Java 有天生优势。而另一个新生代 fluentd 则是标准的 Ruby 产品(即 Matz's Ruby Interpreter)。logstash 选用 JRuby 来实现，似乎有点两头不讨好啊？

乔丹西塞曾经多次著文聊过这个问题。为了避凑字数的嫌，这里罗列他的 gist 地址：

* [Time sucks](https://gist.github.com/jordansissel/2929216) 一文是关于 Time 对象的性能测试，最快的生成方法是 `sprintf` 方法，MRI 性能为 82600 call/sec，JRuby1.6.7 为 131000 call/sec，而 JRuby1.7.0 为 215000 call/sec。
* [Comparing egexp patterns speeds](https://gist.github.com/jordansissel/1491302)
 一文是关于正则表达式的性能测试，使用的正则统一为 `(?-mix:('(?:[^\\']+|(?:\\.)+)*'))`，结果 MRI1.9.2 为 530000 matches/sec，而 JRuby1.6.5 为 690000 matches/sec。
* [Logstash performance under ruby](https://gist.github.com/jordansissel/4171039)一文是关于 logstash 本身数据流转性能的测试，使用 *inputs/generator* 插件生成数据，*outputs/stdout* 到 pv 工具记点统计。结果 MRI1.9.3 为 4000 events/sec，而 JRuby1.7.0 为 25000 events/sec。

可能你已经运行着 logstash 并发现自己的线上数据远超过这个测试——这是因为乔丹西塞在2013年之前，一直是业余时间开发 logstash，而且从未用在自己线上过。所以当时的很多测试是在他自己电脑上完成的。

在 logstash 得到大家强烈关注后，作者发表了《[logstash needs full time love](https://gist.github.com/jordansissel/3088552)》，表明了这点并求一份可以让自己全职开发 logstash 的工作，同时列出了1.1.0 版本以后的 roadmap。（不过事实证明当时作者列出来的这些需求其实不紧急，因为大多数，或者说除了 kibana 以外，至今依然没有==!）

时间轴继续向前推，到 2011 年，你会发现 logstash 原先其实也是用 MRI1.8.7 写的！在 [grok 模块从 C 扩展改写成 FFI 扩展后](https://code.google.com/p/logstash/issues/detail?id=37)，才正式改用 JRuby。

切换语言的当时，乔丹西塞发表了《[logstash, why jruby?](https://gist.github.com/jordansissel/978956)》大家可以一读。

事实上，时至今日，多种 Ruby 实现的痕迹(到处都有 RUBY_ENGINE 变量判断)依然遍布 logstash 代码各处，作者也力图保证尽可能多的代码能在 MRI 上运行。

作为简单的指示，在和插件无关的核心代码中，只有 LogStash::Event 里生成 `@timestamp`字段时用了 Java 的 joda 库为 JRuby 仅有的。稍微修改成 Ruby 自带的 Time 库，即可在 MRI 上运行起来。而主要插件中，也只有 filters/date 和 outputs/elasticsearch 是 Java 相关的。
