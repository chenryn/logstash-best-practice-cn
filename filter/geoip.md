# GeoIP 地址查询归类


## 配置示例

```
filter {
    geoip {
        source => "message"
    }
}
```


## 运行结果

```ruby
{
       "message" => "183.60.92.253",
      "@version" => "1",
    "@timestamp" => "2014-08-07T10:32:55.610Z",
          "host" => "raochenlindeMacBook-Air.local",
         "geoip" => {
                      "ip" => "183.60.92.253",
           "country_code2" => "CN",
           "country_code3" => "CHN",
            "country_name" => "China",
          "continent_code" => "AS",
             "region_name" => "30",
               "city_name" => "Guangzhou",
                "latitude" => 23.11670000000001,
               "longitude" => 113.25,
                "timezone" => "Asia/Chongqing",
        "real_region_name" => "Guangdong",
                "location" => [
            [0] 113.25,
            [1] 23.11670000000001
        ]
    }
}
```
