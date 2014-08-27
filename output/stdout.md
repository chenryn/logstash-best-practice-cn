# 标准输出(Stdout)

和之前 *inputs/stdin* 插件一样，*outputs/stdout* 插件也是最基础和简单的输出插件。同样在这里简单介绍一下，作为输出插件的一个共性了解。

## 配置示例

```
output {
    stdout {
        codec => rubydebug
        workers => 2
    }
}
```

## 解释

输出插件统一具有一个参数是 `workers`。Logstash 为输出做了多线程的准备。

其次是 codec 设置。codec 的作用在之前已经讲过。可能除了 `codecs/multiline` ，其他 codec 插件本身并没有太多的设置项。所以一般省略掉后面的配置区段。换句话说。上面配置示例的完全写法应该是：

```
output {
    stdout {
        codec => rubydebug {
        }
        workers => 2
    }
}
```

单就 *outputs/stdout* 插件来说，其最重要和常见的用途就是调试。所以在不太有效的时候，加上命令行参数 `-vv` 运行，查看更多详细调试信息。
