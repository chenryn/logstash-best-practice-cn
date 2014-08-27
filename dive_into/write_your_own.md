# 自己写一个插件

前面已经提过在运行 logstash 的时候，可以通过 `--pluginpath` 参数来加载自己写的插件。那么，插件又该怎么写呢？

## 插件格式

一个标准的 logstash 输入插件格式如下：

```
require 'logstash/namespace'
require 'logstash/inputs/base'
class LogStash::Inputs::MyPlugin < LogStash::Inputs::Base
  config_name 'myplugin'
  milestone 1
  config :myoption_key, :validate => :string, :default => 'myoption_value'
  public def register
  end
  public def run(queue)
  end
end
```

其中大多数语句在过滤器和输出阶段是共有的。

* config_name 用来定义该插件写在 logstash 配置文件里的名字；
* milestone 标记该插件的开发里程碑，一般为1，2，3，如果不再维护的，标记为 0；
* config 可以定义很多个，即该插件在 logstash 配置文件中的可配置参数。logstash 很温馨的提供了验证方法，确保接收的数据是你期望的数据类型；
* register logstash 在启动的时候运行的函数，一些需要常驻内存的数据，可以在这一步先完成。比如对象初始化，*filters/ruby* 插件中的 `init` 语句等。

输入插件独有的是 run 方法。在 run 方法中，必须实现一个长期运行的程序(最简单的就是 loop 指令)。然后在每次收到数据并处理成 `event` 之后，一定要调用 `queue << event` 语句。一个输入流程就算是完成了。

而如果是过滤器插件，对应修改成：

```
require 'logstash/filters/base'
class LogStash::Filters::MyPlugin < LogStash::Filters::Base
  public def filter(event)
  end
end
```

输出插件则是：

```
require 'logstash/outputs/base'
class LogStash::Outputs::MyPlugin < LogStash::Outputs::Base
  public def receive(event)
  end
end
```

另外，为了在终止进程的时候不遗失数据，建议都实现如下这个方法，只要实现了，logstash 在 shutdown 的时候就会自动调用：

```
public def teardown
end
```

## 推荐阅读

* [Extending logstash](http://logstash.net/docs/1.4.2/extending/)
* [Plugin Milestones](http://logstash.net/docs/1.4.2/plugin-milestones)
