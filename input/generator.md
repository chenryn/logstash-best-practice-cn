# 生成测试数据(Generator)

实际运行的时候这个插件是派不上用途的，但这个插件依然是非常重要的插件之一。因为每一个使用 ELK stack 的运维人员都应该清楚一个道理：数据是支持操作的唯一真理（否则你也用不着 ELK）。所以在上线之前，你一定会需要在自己的实际环境中，测试 Logstash 和 Elasticsearch 的性能状况。这时候，这个用来生成测试数据的插件就有用了！

## 配置示例

```
input {
    generator {
        count => 10000000
        message => '{"key1":"value1","key2":[1,2],"key3":{"subkey1":"subvalue1"}}'
        codec => json
    }
}
```

插件的默认生成数据，message 内容是 "hello world"。你可以根据自己的实际需要这里来写其他内容。

## 使用方式

做测试有两种主要方式：

* 配合 LogStash::Outputs::Null

inputs/generator 是无中生有，output/null 则是锯嘴葫芦。事件流转到这里直接就略过，什么操作都不做。相当于只测试 Logstash 的 pipe 和 filter 效率。测试过程非常简单：

    $ time ./bin/logstash -f generator_null.conf
    real	3m0.864s
    user	3m39.031s
    sys		0m51.621s

* 使用 pv 命令配合 LogStash::Outputs::Stdout 和 LogStash::Codecs::Dots

上面的这种方式虽然想法挺好，不过有个小漏洞：logstash 是在 JVM 上运行的，有一个明显的启动时间，运行也有一段事件的预热后才算稳定运行。所以，要想更真实的反应 logstash 在长期运行时候的效率，还有另一种方法：

```
output {
    stdout {
        codec => dots
    }
}
```

LogStash::Codecs::Dots 也是一个另类的 codec 插件，他的作用是：把每个 event 都变成一个点(`.`)。这样，在输出的时候，就变成了一个一个的 `.` 在屏幕上。显然这也是一个为了测试而存在的插件。

下面就要介绍 pv 命令了。这个命令的作用，就是作实时的标准输入、标准输出监控。我们这里就用它来监控标准输出：

    $ ./bin/logstash -f generator_dots.conf | pv -abt > /dev/null
    2.2MiB 0:03:00 [12.5kiB/s]

可以很明显的看到在前几秒中，速度是 0 B/s，因为 JVM 还没启动起来呢。开始运行的时候，速度依然不快。慢慢增长到比较稳定的状态，这时候的才是你需要的数据。

这里单位是 B/s，但是因为一个 event 就输出一个 `.`，也就是 1B。所以 12.5kiB/s 就相当于是 12.5k event/s。

*注：如果你在 CentOS 上通过 yum 安装的 pv 命令，版本较低，可能还不支持 -a 参数。单纯靠 -bt 参数看起来还是有点累的。*

## 额外的话

既然单独花这么一节来说测试，这里想额外谈谈一个很常见的话题：* ELK 的性能怎么样？*

**其实这压根就是一个不正确的提问**。ELK 并不是一个软件而是一个并不耦合的套件。所以，我们需要分拆开讨论这三个软件的性能如何？怎么优化？

* LogStash 的性能，是最让新人迷惑的地方。因为 LogStash 本身并不维护队列，所以整个日志流转中任意环节的问题，都可能看起来像是 LogStash 的问题。这里，需要熟练使用本节说的测试方法，针对自己的每一段配置，都确定其性能。另一方面，就是本书之前提到过的，LogStash 给自己的线程都设置了单独的线程名称，你可以在 `top -H` 结果中查看具体线程的负载情况。

* Elasticsearch 的性能。这里最需要强调的是：Elasticsearch 是一个分布式系统。从来没有分布式系统要跟人比较单机处理能力的说法。所以，更需要关注的是：在确定的单机处理能力的前提下，性能是否能做到线性扩展。当然，这不意味着说提高处理能力只能靠加机器了——有效利用 mapping API 是非常重要的。不过暂时就在这里讲述了。

* Kibana 的性能。通常来说，Kibana 只是一个单页 Web 应用，只需要 nginx 发布静态文件即可，没什么性能问题。页面加载缓慢，基本上是因为 Elasticsearch 的请求响应时间本身不够快导致的。不过一定要细究的话，也能找出点 Kibana 本身性能相关的话题：因为 Kibana3 默认是连接固定的一个 ES 节点的 IP 端口的，所以这里会涉及一个浏览器的同一 IP 并发连接数的限制。其次，就是 Kibana 用的 AngularJS 使用了 Promise.then 的方式来处理 HTTP 请求响应。这是异步的。
