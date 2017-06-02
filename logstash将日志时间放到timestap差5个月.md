### logstash将时间放到timestamp去，居然发现时间差五个月
```
input { stdin { } } 
filter{
    grok{
	patterns_dir => "/elkr5.3.2/logstash5/patterns"
 	match => [
      		"message","%{HIST}"
        ]
    }
    if "_grokparsefailure" in [tags] {
          drop {}
    }
	mutate{
            convert => ["cmdtime","string"]
    }
	date {
			match => ["cmdtime","YYYY-MM-DD HH:mm:ss"] 
			target => "@timestamp"
			add_tag => [ "tmatch" ]
	}
	mutate {
		remove_field => ["cmdtime"]
	}	
}
output { stdout { codec => rubydebug } }
```
日志格式如下
```
2017-06-02 09:00:33 root cat /var/log/.history.log
```
最初显示结果如下
```
[root@localhost elkr5.3.2]# ./1
 [+]testing 
Sending Logstash's logs to /elkr5.3.2/logstash5/logs/ which is now configured via log4j2.properties
[2017-06-02T09:47:52,713][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>32, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>4000}
[2017-06-02T09:47:52,772][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
[2017-06-02T09:47:52,820][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9601}
2017-06-02 09:00:33 root cat /var/log/.history.log
{
    "@timestamp" => 2017-01-02T01:00:33.000Z,
       "cmdtime" => "2017-06-02 09:00:33",
      "@version" => "1",
          "host" => "localhost.localdomain",
           "cmd" => "cat /var/log/.history.log",
       "message" => "2017-06-02 09:00:33 root cat /var/log/.history.log",
          "user" => "root",
          "tags" => [
        [0] "tmatch"
    ]
}
```
查到[logstash手册](https://www.elastic.co/guide/en/logstash/5.3/plugins-filters-date.html)的时候，才发现原来是这里写错了。

```
match => ["cmdtime","YYYY-MM-DD HH:mm:ss"]
```

正确的应该是
```
match => ["cmdtime","YYYY-MM-dd HH:mm:ss"]
```
正确替换后，差8个小时无所谓，kibana会自动解决时区问题。
```
[root@localhost elkr5.3.2]# ./1
 [+]testing 
Sending Logstash's logs to /elkr5.3.2/logstash5/logs/ which is now configured via log4j2.properties
[2017-06-02T09:55:51,472][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>32, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>4000}
[2017-06-02T09:55:51,529][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
[2017-06-02T09:55:51,580][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9601}
2017-06-02 09:00:33 root cat /var/log/.history.log
{
    "@timestamp" => 2017-06-02T01:00:33.000Z,
       "cmdtime" => "2017-06-02 09:00:33",
      "@version" => "1",
          "host" => "localhost.localdomain",
           "cmd" => "cat /var/log/.history.log",
       "message" => "2017-06-02 09:00:33 root cat /var/log/.history.log",
          "user" => "root",
          "tags" => [
        [0] "tmatch"
    ]
}
```

当时是从别人那里抄来的，遇到问题还是要多查手册。
