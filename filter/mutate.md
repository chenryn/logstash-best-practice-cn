# 数据修改(Mutate)

*filters/mutate* 插件是 Logstash 另一个重要插件。它提供了丰富的基础类型数据处理能力。包括类型转换，字符串处理和字段处理等。

## 类型转换

类型转换是 *filters/mutate* 插件最初诞生时的唯一功能。其应用场景在之前 [Codec/JSON](../codec/json.md) 小节已经提到。

可以设置的转换类型包括："integer"，"float" 和 "string"。示例如下：

```
filter {
    mutate {
        convert => ["request_time", "float"]
    }
}
```

**注意：mutate 除了转换简单的字符值，还支持对数组类型的字段进行转换，即将 `["1","2"]` 转换成 `[1,2]`。但不支持对哈希类型的字段做类似处理。有这方面需求的可以采用稍后讲述的 filters/ruby 插件完成。**

## 字符串处理

* gsub

仅对字符串类型字段有效

```
    gsub => ["urlparams", "[\\?#]", "_"]
```

* split

```
filter {
    mutate {
        split => ["message", "|"]
    }
}
```

随意输入一串以`|`分割的字符，比如 "123|321|adfd|dfjld*=123"，可以看到如下输出：

```ruby
{
    "message" => [
        [0] "123",
        [1] "321",
        [2] "adfd",
        [3] "dfjld*=123"
    ],
    "@version" => "1",
    "@timestamp" => "2014-08-20T15:58:23.120Z",
    "host" => "raochenlindeMacBook-Air.local"
}
```

* join

仅对数组类型字段有效

我们在之前已经用 `split` 割切的基础再 `join` 回去。配置改成：

```
filter {
    mutate {
        split => ["message", "|"]
    }
    mutate {
        join => ["message", ","]
    }
}
```

filter 区段之内，是顺序执行的。所以我们最后看到的输出结果是：

```ruby
{
    "message" => "123,321,adfd,dfjld*=123",
    "@version" => "1",
    "@timestamp" => "2014-08-20T16:01:33.972Z",
    "host" => "raochenlindeMacBook-Air.local"
}
```

* merge

合并两个数组或者哈希字段。依然在之前 split 的基础上继续：

```
filter {
    mutate {
        split => ["message", "|"]
    }
    mutate {
        merge => ["message", "message"]
    }
}
```

我们会看到输出：

```ruby
{
       "message" => [
        [0] "123",
        [1] "321",
        [2] "adfd",
        [3] "dfjld*=123",
        [4] "123",
        [5] "321",
        [6] "adfd",
        [7] "dfjld*=123"
    ],
      "@version" => "1",
    "@timestamp" => "2014-08-20T16:05:53.711Z",
          "host" => "raochenlindeMacBook-Air.local"
}
```

如果 src 字段是字符串，会自动先转换成一个单元素的数组再合并。把上一示例中的来源字段改成 "host"：

```
filter {
    mutate {
        split => ["message", "|"]
    }
    mutate {
        merge => ["message", "host"]
    }
}
```

结果变成：

```ruby
{
       "message" => [
        [0] "123",
        [1] "321",
        [2] "adfd",
        [3] "dfjld*=123",
        [4] "raochenlindeMacBook-Air.local"
    ],
      "@version" => "1",
    "@timestamp" => "2014-08-20T16:07:53.533Z",
          "host" => [
        [0] "raochenlindeMacBook-Air.local"
    ]
}
```

看，目的字段 "message" 确实多了一个元素，但是来源字段 "host" 本身也由字符串类型变成数组类型了！

下面你猜，如果来源位置写的不是字段名而是直接一个字符串，会产生什么奇特的效果呢？

* strip
* lowercase
* uppercase


## 字段处理

* rename

重命名某个字段，如果目的字段已经存在，会被覆盖掉：

```
filter {
    mutate {
        rename => ["syslog_host", "host"]
    }
}
```

* update

更新某个字段的内容。如果字段不存在，不会新建。


* replace

作用和 update 类似，但是当字段不存在的时候，它会起到 `add_field` 参数一样的效果，自动添加新的字段。


## 执行次序

需要注意的是，filter/mutate 内部是有执行次序的。其次序如下：

```
    rename(event) if @rename
    update(event) if @update
    replace(event) if @replace
    convert(event) if @convert
    gsub(event) if @gsub
    uppercase(event) if @uppercase
    lowercase(event) if @lowercase
    strip(event) if @strip
    remove(event) if @remove
    split(event) if @split
    join(event) if @join
    merge(event) if @merge

    filter_matched(event)
```

而 `filter_matched` 这个 filters/base.rb 里继承的方法也是有次序的。

```
  @add_field.each do |field, value|
  end
  @remove_field.each do |field|
  end
  @add_tag.each do |tag|
  end
  @remove_tag.each do |tag|
  end
```
