# 数值统计(Metrics)

*filters/metrics* 插件是使用 Ruby 的 *Metriks* 模块来实现在内存里实时的计数和采样分析。该模块支持两个类型的数值分析：meter 和 timer。下面分别举例说明：

## Meter 示例(速率阈值检测)

web 访问日志的异常状态码频率是运维人员会非常关心的一个数据。通常我们的做法，是通过 logstash 或者其他日志分析脚本，把计数发送到 rrdtool 或者 graphite 里面。然后再通过 check_graphite 脚本之类的东西来检查异常并报警。

事实上这个事情可以直接在 logstash 内部就完成。比如如果最近一分钟 504 请求的个数超过 100 个就报警：

```
filter {
    metrics {
        meter => "error.%{status}"
        add_tag => "metric"
        ignore_older_than => 10
    }
    if "metric" in [tags] {
        ruby {
            code => "event.cancel if event['error.504.rate_1m'] * 60 < 100"
        }
    }
}
output {
    if "metric" in [tags] {
        exec {
            command => "echo \"Out of threshold: %{error.504.rate_1m}\""
        }
    }
}
```

这里需要注意 `*60` 的含义。

metriks 模块生成的 *rate_1m/5m/15m* 意思是：最近 1，5，15 分钟的**每秒**速率！


## Timer 示例(box and whisker 异常检测)

官版的 *filters/metrics* 插件只适用于 metric 事件的检查。由插件生成的新事件内部不存有来自 input 区段的实际数据信息。所以，要完成我们的百分比分布箱体检测，需要首先对代码稍微做几行变动，即在 metric 的 timer  事件里加一个属性，存储最近一个实际事件的数值：<https://github.com/chenryn/logstash/commit/bc7bf34caf551d8a149605cf28e7c5d33fae7458>

然后我们就可以用如下配置来探测异常数据了：

```
filter {
    metrics {
        timer => {"rt" => "%{request_time}"}
        percentiles => [25, 75]
        add_tag => "percentile"
    }
    if "percentile" in [tags] {
        ruby {
            code => "l=event['rt.p75']-event['rt.p25'];event['rt.low']=event['rt.p25']-l;event['rt.high']=event['rt.p75']+l"
        }
    }
}
output {
    if "percentile" in [tags] and ([rt.last] > [rt.high] or [rt.last] < [rt.low]) {
        exec {
            command => "echo \"Anomaly: %{rt.last}\""
        }
    }
}
```

*小贴士：有关 box and shisker plot 内容和重要性，参见《数据之魅》一书。*
