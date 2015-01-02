# split 拆分事件

上一章我们通过 multiline 插件将多行数据合并进一个事件里，那么反过来，也可以把一行数据，拆分成多个事件。这就是 split 插件。

## 配置示例

```
filter {
    split {
        field => "message"
        terminator => "#"
    }
}
```

## 运行结果

这个测试中，我们在 intputs/stdin 的终端中输入一行数据："test1#test2"，结果看到输出两个事件：

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "test1"
}
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "test2"
}
```

## 重要提示

split 插件中使用的是 yield 功能，其结果是 split 出来的新事件，会直接结束其在 filter 阶段的历程，也就是说写在 split 后面的其他 filter 插件都不起作用，进入到 output 阶段。所以，一定要保证 **split 配置写在全部 filter 配置的最后**。

使用了类似功能的还有 clone 插件。

*注：从 logstash-1.5.0beta1 版本以后修复该问题。*
