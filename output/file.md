# 保存成文件(File)

通过日志收集系统将分散在数百台服务器上的数据集中存储在某中心服务器上，这是运维最原始的需求。早年的 scribed ，甚至直接就把输出的语法命名为 `<store>`。Logstash 当然也能做到这点。

和 `LogStash::Inputs::File` 不同, `LogStash::Outputs::File` 里可以使用 sprintf format 格式来自动定义输出到带日期命名的路径。

## 配置示例

```
output {
    file {
        path => "/path/to/%{+yyyy/MM/dd/HH}/%{host}.log.gz"
        message_format => "%{message}"
        gzip => true
    }
}
```

## 解释

使用 *output/file* 插件首先需要注意的就是 `message_format` 参数。插件默认是输出整个 event 的 JSON 形式数据的。这可能跟大多数情况下使用者的期望不符。大家可能只是希望按照日志的原始格式保存就好了。所以需要定义为 `%{message}`，当然，前提是在之前的 *filter* 插件中，你没有使用 `remove_field` 或者 `update` 等参数删除或修改 `%{message}` 字段的内容。

另一个非常有用的参数是 gzip。gzip 格式是一个非常奇特而友好的格式。其格式包括有：

* 10字节的头，包含幻数、版本号以及时间戳
* 可选的扩展头，如原文件名
* 文件体，包括DEFLATE压缩的数据
* 8字节的尾注，包括CRC-32校验和以及未压缩的原始数据长度

这样 gzip 就可以一段一段的识别出来数据 —— **反过来说，也就是可以一段一段压缩了添加在后面！**

这对于我们流式添加数据简直太棒了！

*小贴士：你或许见过网络流传的 parallel 命令行工具并发处理数据的神奇文档，但在自己用的时候总见不到效果。实际上就是因为：文档中处理的 gzip 文件，可以分开处理然后再合并的。*


