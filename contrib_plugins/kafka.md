# Kafka

<https://github.com/joekiller/logstash-kafka>

插件已经正式合并进官方仓库，以下使用介绍基于**logstash 1.4相关版本**，1.5及以后版本的使用后续依照官方文档持续更新。

插件本身内容非常简单，其主要依赖同一作者写的 [jruby-kafka](https://github.com/joekiller/jruby-kafka) 模块。需要注意的是：**该模块仅支持 Kafka－0.8 版本。如果是使用 0.7 版本 kafka 的，将无法直接使 jruby-kafka 该模块和 logstash-kafka 插件。**

## 安装

* 安装按照官方文档完全自动化的安装.或是可以通过以下方式手动自己安装插件，不过重点注意的是 **kafka 的版本**，上面已经指出了。

> 1. 下载 logstash 并解压重命名为 `./logstash-1.4.0` 文件目录。

> 2. 下载 kafka 相关组件，以下示例选的为 [kafka_2.8.0-0.8.1.1-src](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.8.1.1/kafka-0.8.1.1-src.tgz)，并解压重命名为 `./kafka_2.8.0-0.8.1.1`。

> 3. 下载 logstash-kafka v0.4.2 从 [releases](https://github.com/joekiller/logstash-kafka/releases)，并解压重命名为 `./logstash-kafka-0.4.2`。

> 4. 从 `./kafka_2.8.0-0.8.1.1/libs` 目录下复制所有的 jar 文件拷贝到 `./logstash-1.4.0/vendor/jar/kafka_2.8.0-0.8.1.1/libs` 下，其中你需要创建 `kafka_2.8.0-0.8.1.1/libs` 相关文件夹及目录。

> 5. 分别复制 `./logstash-kafka-0.4.2/logstash` 里的 `inputs` 和 `outputs` 下的 `kafka.rb`，拷贝到对应的 `./logstash-1.4.0/lib/logstash` 里的 `inputs` 和 `outputs` 对应目录下。

> 6. 切换到 `./logstash-1.4.0` 目录下，现在需要运行 logstash-kafka 的 gembag.rb 脚本去安装 jruby-kafka 库，执行以下命令： `GEM_HOME=vendor/bundle/jruby/1.9 GEM_PATH= java -jar vendor/jar/jruby-complete-1.7.11.jar --1.9 ../logstash-kafka-0.4.2/gembag.rb ../logstash-kafka-0.4.2/logstash-kafka.gemspec`。

> 7. 现在可以使用 logstash-kafka 插件运行 logstash 了。例如：`bin/logstash agent -f logstash.conf`。

## Input 配置示例

以下配置可以实现对 kafka 读取端(consumer)的基本使用。

消费端更多详细的配置请查看 <http://kafka.apache.org/documentation.html#consumerconfigs> kafka 官方文档的消费者部分配置文档。

```
input {
    kafka {
        zk_connect => "localhost:2181"
        group_id => "logstash"
        topic_id => "test"
        reset_beginning => false # boolean (optional)， default: false
        consumer_threads => 5  # number (optional)， default: 1
        decorate_events => true # boolean (optional)， default: false
        }
    }
```

## Input 解释

消费端的一些比较有用的配置项：

* group_id

消费者分组，可以通过组 ID 去指定，不同的组之间消费是相互不受影响的，相互隔离。

* topic_id

指定消费话题，也是必填项目，指定消费某个 `topic` ，这个其实就是订阅某个主题，然后去消费。

* reset_beginning

logstash 启动后从什么位置开始读取数据，默认是结束位置，也就是说 logstash 进程会以从上次读取结束时的偏移量开始继续读取，如果之前没有消费过，那么就开始从头读取.如果你是要导入原有数据，把这个设定改成 "true"， logstash 进程就从头开始读取.有点类似 `cat` ，但是读到最后一行不会终止，而是变成 `tail -F` ，继续监听相应数据。

* decorate_events

在输出消息的时候会输出自身的信息包括:消费消息的大小， topic 来源以及 consumer 的 group 信息。

* rebalance\_max\_retries

当有新的 consumer(logstash) 加入到同一 group 时，将会 `reblance` ，此后将会有 `partitions` 的消费端迁移到新的 `consumer` 上，如果一个 `consumer` 获得了某个 `partition` 的消费权限，那么它将会向 `zookeeper` 注册， `Partition Owner registry` 节点信息，但是有可能此时旧的 `consumer` 尚没有释放此节点，此值用于控制，注册节点的重试次数。

* consumer\_timeout\_ms

指定时间内没有消息到达就抛出异常，一般不需要改。

以上是相对重要参数的使用示例，更多参数可以选项可以跟据 <https://github.com/joekiller/logstash-kafka/blob/master/README.md> 查看 input 默认参数。

## 注意

1.想要使用多个 logstash 端协同消费同一个 `topic` 的话，那么需要把两个或是多个 logstash 消费端配置成相同的 `group_id` 和 `topic_id`， 但是前提是要把**相应的 topic 分多个 partitions (区)**，多个消费者消费是无法保证消息的消费顺序性的。

> 这里解释下，为什么要分多个 **partitions(区)**， kafka 的消息模型是对 topic 分区以达到分布式效果。每个 `topic` 下的不同的 **partitions (区)**只能有一个 **Owner** 去消费。所以只有多个分区后才能启动多个消费者，对应不同的区去消费。其中协调消费部分是由 server 端协调而成。不必使用者考虑太多。只是**消息的消费则是无序的**。

总结:保证消息的顺序，那就用一个 **partition**。 **kafka 的每个 partition 只能同时被同一个 group 中的一个 consumer 消费**。

## Output 配置

以下配置可以实现对 kafka 写入端 (producer) 的基本使用。

生产端更多详细的配置请查看 <http://kafka.apache.org/documentation.html#producerconfigs> kafka 官方文档的生产者部分配置文档。

```
 output {
    kafka {
        broker_list => "localhost:9092"
        topic_id => "test"
        compression_codec => "snappy" # string (optional)， one of ["none"， "gzip"， "snappy"]， default: "none"
    }
}
```

## Output 解释

生产的可设置性还是很多的，设置其实更多，以下是更多的设置：

* compression_codec

消息的压缩模式，默认是 none，可以有 gzip 和 snappy (暂时还未测试开启压缩与不开启的性能，数据传输大小等对比)。

* compressed_topics

可以针对特定的 topic 进行压缩，设置这个参数为 `topic` ，表示此 `topic` 进行压缩。

* request\_required\_acks

消息的确认模式:

> 可以设置为 0: 生产者不等待 broker 的回应，只管发送.会有最低能的延迟和最差的保证性(在服务器失败后会导致信息丢失)
>
> 可以设置为 1: 生产者会收到 leader 的回应在 leader 写入之后.(在当前 leader 服务器为复制前失败可能会导致信息丢失)
>
> 可以设置为 -1: 生产者会收到 leader 的回应在全部拷贝完成之后。

* partitioner_class

分区的策略，默认是 hash 取模

* send\_buffer\_bytes

socket 的缓存大小设置，其实就是缓冲区的大小

#### 消息模式相关

* serializer_class

消息体的系列化处理类，转化为字节流进行传输，**请注意 encoder 必须和下面的 `key_serializer_class` 使用相同的类型**。

* key\_serializer\_class

默认的是与 `serializer_class` 相同

* producer_type

生产者的类型 `async` 异步执行消息的发送 `sync` 同步执行消息的发送

* queue\_buffering\_max\_ms

**异步模式下**，那么就会在设置的时间缓存消息，并一次性发送

* queue\_buffering\_max\_messages

**异步的模式下**，最长等待的消息数

* queue\_enqueue\_timeout\_ms

**异步模式下**，进入队列的等待时间，若是设置为0，那么要么进入队列，要么直接抛弃

* batch\_num\_messages

**异步模式下**，每次发送的最大消息数，前提是触发了 `queue_buffering_max_messages` 或是 `queue_enqueue_timeout_ms` 的限制

以上是相对重要参数的使用示例，更多参数可以选项可以跟据 <https://github.com/joekiller/logstash-kafka/blob/master/README.md> 查看 output 默认参数。

### 小贴士

默认情况下，插件是使用 json 编码来输入和输出相应的消息，消息传递过程中 logstash 默认会为消息编码内加入相应的时间戳和 hostname 等信息。如果不想要以上信息(一般做消息转发的情况下)，可以使用以下配置，例如:

```
 output {
    kafka {
        codec => plain {
            format => "%{message}"
        }
    }
}
```
