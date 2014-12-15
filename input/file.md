# 读取文件(File)

分析网站访问日志应该是一个运维工程师最常见的工作了。所以我们先学习一下怎么用 logstash 来处理日志文件。

Logstash 使用一个名叫 *FileWatch* 的 Ruby Gem 库来监听文件变化。这个库支持 glob 展开文件路径，而且会记录一个叫 *.sincedb* 的数据库文件来跟踪被监听的日志文件的当前读取位置。所以，不要担心 logstash 会漏过你的数据。

*sincedb 文件中记录了每个被监听的文件的 inode, major number, minor number 和 pos。*

## 配置示例

```
input
    file {
        path => ["/var/log/*.log", "/var/log/message"]
        type => "system"
        start_position => "beginning"
    }
}
```

## 解释

有一些比较有用的配置项，可以用来指定 *FileWatch* 库的行为：

* discover_interval

logstash 每隔多久去检查一次被监听的 `path` 下是否有新文件。默认值是 15 秒。

* exclude

不想被监听的文件可以排除出去，这里跟 `path` 一样支持 glob 展开。

* sincedb_path

如果你不想用默认的 `$HOME/.sincedb`，可以通过这个配置定义 sincedb 文件到其他位置。

* sincedb_write_interval

logstash 每隔多久写一次 sincedb 文件，默认是 15 秒。

* stat_interval

logstash 每隔多久检查一次被监听文件状态（是否有更新），默认是 1 秒。

* start_position

logstash 从什么位置开始读取文件数据，默认是结束位置，也就是说 logstash 进程会以类似 `tail -F` 的形式运行。如果你是要导入原有数据，把这个设定改成 "beginning"，logstash 进程就从头开始读取，有点类似 `cat`，但是读到最后一行不会终止，而是继续变成 `tail -F`。

## 注意

1. 通常你要导入原有数据进 Elasticsearch 的话，你还需要 [filter/date](../filter/date.md) 插件来修改默认的"@timestamp" 字段值。稍后会学习这方面的知识。
2. *FileWatch* 只支持文件的**绝对路径**，而且会不自动递归目录。所以有需要的话，请用数组方式都写明具体哪些文件。
3. *LogStash::Inputs::File* 只是在进程运行的注册阶段初始化一个 *FileWatch* 对象。所以它不能支持类似 fluentd 那样的 `path => "/path/to/%{+yyyy/MM/dd/hh}.log"` 写法。达到相同目的，你只能写成 `path => "/path/to/*/*/*/*.log"`。
4. `start_position` 仅在该文件从未被监听过的时候起作用。如果 sincedb 文件中已经有这个文件的 inode 记录了，那么 logstash 依然会从记录过的 pos 开始读取数据。
5. 因为 windows 平台上没有 inode 的概念，Logstash 某些版本在 windows 平台上监听文件不是很靠谱。windows 平台上，推荐考虑使用 nxlog 作为收集端，参阅本书[稍后](../ecosystem/nxlog.md)章节。
