# HDFS



* <https://github.com/dstore-dbap/logstash-webhdfs>

This plugin based on WebHDFS api of Hadoop, it just POST data to WebHDFS port. So, it's a native Ruby code.

```
output {
    hadoop_webhdfs {
        workers => 2
        server => "your.nameno.de:14000"
        user => "flume"
        path => "/user/flume/logstash/dt=%{+Y}-%{+M}-%{+d}/logstash-%{+H}.log"
        flush_size => 500
        compress => "snappy"
        idle_flush_time => 10
        retry_interval => 0.5
    }
}
```


* <https://github.com/avishai-ish-shalom/logstash-hdfs>

This plugin based on HDFS api of Hadoop, it import java classes like `org.apache.hadoop.fs.FileSystem` etc.

### Configuration

```
output {
    hdfs {
        path => "/path/to/output_file.log"
        enable_append => true
    }
}
```

### Howto run

```
CLASSPATH=$(find /path/to/hadoop -name '*.jar' | tr '\n' ':'):/etc/hadoop/conf:/path/to/logstash-1.1.7-monolithic.jar java logstash.runner agent -f conf/hdfs-output.conf -p /path/to/cloned/logstash-hdfs
```
