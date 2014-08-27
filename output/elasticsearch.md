# 保存进 Elasticsearch

Logstash 早期有三个不同的 elasticsearch 插件。到 1.4.0 版本的时候，开发者彻底重写了 `LogStash::Outputs::Elasticsearch` 插件。从此，我们只需要用这一个插件，就能任意切换使用 Elasticsearch 集群支持的各种不同协议了。

## 配置示例

```
output {
    elasticsearch {
        host => "192.168.0.2"
        protocol => "http"
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
        index_type => "%{type}"
        workers => 5
    }
}
```

## 解释

### 协议

现在，新插件支持三种协议： *node*，*http* 和 *transport*。

一个小集群里，使用 *node* 协议最方便了。Logstash 以 elasticsearch 的 client 节点身份(即不存数据不参加选举)运行。如果你运行下面这行命令，你就可以看到自己的 logstash 进程名，对应的 `node.role` 值是 **c**：

```
# curl 127.0.0.1:9200/_cat/nodes?v
host       ip      heap.percent ram.percent load node.role master name
local 192.168.0.102  7      c         -      logstash-local-1036-2012
local 192.168.0.2    7      d         *      Sunstreak

```

特别的，作为一个快速运行示例的需要，你还可以在 logstash 进程内部运行一个**内嵌**的 elasticsearch 服务器。内嵌服务器默认会在 `$PWD/data` 目录里存储索引。如果你想变更这些配置，在 `$PWD/elasticsearch.yml` 文件里写自定义配置即可，logstash 会尝试自动加载这个文件。

对于拥有很多索引的大集群，你可以用 *transport* 协议。logstash 进程会转发所有数据到你指定的某台主机上。这种协议跟上面的 *node* 协议是不同的。*node* 协议下的进程是可以接收到整个 Elasticsearch 集群状态信息的，当进程收到一个事件时，它就知道这个事件应该存在集群内哪个机器的分片里，所以它就会直接连接该机器发送这条数据。而 *transport* 协议下的进程不会保存这个信息，在集群状态更新(节点变化，索引变化都会发送全量更新)时，就不会对所有的 logstash 进程也发送这种信息。更多 Elasticsearch 集群状态的细节，参阅<http://www.elasticsearch.org/guide>。

*小贴士：官版的 transport 协议下也没有了自动往集群多台机器发数据的负载均衡能力。*

如果你已经有现成的 Elasticsearch 集群，但是版本跟 logstash 自带的又不太一样，建议你使用 *http* 协议。Logstash 会使用 POST 方式发送数据。注意：Logstash 其实只是在一开始的时候连接你指定的主机，获取节点信息。之后它会从节点列表中选取不同的节点发送数据，达到负载均衡的效果。

*小贴士：开发者在 IRC freenode#logstash 频道里表示："高于 1.0 版本的 Elasticsearch 应该都能跟最新版 logstash 的 node 协议一起正常工作"。此信息仅供参考，请认真测试后再上线。*

### 模板

Elasticsearch 支持给索引预定义设置和 mapping(前提是你用的 elasticsearch 版本支持这个 API，不过估计应该都支持)。Logstash 自带有一个优化好的模板，内容如下:

```json
{
  "template" : "logstash-*",
  "settings" : {
    "index.refresh_interval" : "5s"
  },
  "mappings" : {
    "_default_" : {
       "_all" : {"enabled" : true},
       "dynamic_templates" : [ {
         "string_fields" : {
           "match" : "*",
           "match_mapping_type" : "string",
           "mapping" : {
             "type" : "string", "index" : "analyzed", "omit_norms" : true,
               "fields" : {
                 "raw" : {"type": "string", "index" : "not_analyzed", "ignore_above" : 256}
               }
           }
         }
       } ],
       "properties" : {
         "@version": { "type": "string", "index": "not_analyzed" },
         "geoip"  : {
           "type" : "object",
             "dynamic": true,
             "path": "full",
             "properties" : {
               "location" : { "type" : "geo_point" }
             }
         }
       }
    }
  }
}
```

这其中的关键设置包括：

* index-pattern

只有匹配 `logstash-*` 的索引才会应用这个模板。有时候我们会变更 Logstash 的默认索引名称，记住你也得通过 PUT 方法上传可以匹配你自定义索引名的模板。当然，我更建议的做法是，把你自定义的名字放在 "logstash-" 后面，变成 `index => "logstash-custom-%{+yyyy.MM.dd}"` 这样。

* refresh_interval

Elasticsearch 是一个*近*实时搜索引擎。它实际上是每 1 秒钟刷新一次数据。对于日志分析应用，我们用不着这么实时，所以 logstash 自带的模板修改成了 5 秒钟。你还可以根据需要继续放大这个刷新间隔以提高数据写入性能。

* multi-field with not_analyzed

Elasticsearch 会自动使用自己的默认分词器(空格，点，斜线等分割)来分析字段。分词器对于搜索和评分是非常重要的，但是大大降低了索引写入和聚合请求的性能。所以 logstash 模板定义了一种叫"多字段"(multi-field)类型的字段。这种类型会自动添加一个 ".raw" 结尾的字段，并给这个字段设置为不启用分词器。简单说，你想获取 url 字段的聚合结果的时候，不要直接用 "url" ，而是用 "url.raw" 作为字段名。

* geo_point

Elasticsearch 支持 *geo_point* 类型， *geo distance* 聚合等等。比如说，你可以请求某个 *geo_point* 点方圆 10 千米内数据点的总数。在 Kibana 的 bettermap 类型面板里，就会用到这个类型的数据。

## 推荐阅读

* <http://www.elasticsearch.org/guide>
