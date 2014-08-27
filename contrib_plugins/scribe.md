# scribe


<https://github.com/EverythingMe/logstash-scribeinput>

```
input {
        scribe {
                host => "localhost"
                port => 8000
        }
}
```


```
java -Xmx400M -server \
   -cp scribe_server.jar:logstash-1.2.1-flatjar.jar \
   logstash.runner agent \
   -p /where/did/i/put/this/downloaded/plugin \
   -f logstash.conf
```
