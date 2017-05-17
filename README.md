<font face="微软雅黑"> 

## 《ELKR STACK从入门到放弃》   
ELKR Stack 日志监控平台     
 ![image](https://raw.githubusercontent.com/n3uz/elkr-stack/master/kibana.jpg)
### 一、背景        
日志多，零散。统一集中分析，挖掘攻击事件。自己去编吧

###  二、架构与流程     
![image](https://github.com/n3uz/elkr-stack/blob/master/ELKR%E6%9E%B6%E6%9E%84%E5%9B%BE%E4%B8%8E%E6%B5%81%E7%A8%8B.png?raw=true) 

- Nginx 监听0.0.0.0 80 #配置htpasswd身份认证方式  
- Logstash 监听0.0.0.0:8001 #负责接收各个Beat发送的数据   
- Kibana 监听127.0.0.1:5601 #本地监听，不安装x-pack   
- Redis 监听127.0.0.1:6379  #因为不需要对外开放，可以不设置认证   
- ES 监听127.0.0.1:9200  #集群节点先配置一个，后期再扩展成集群方式  [集群部署方案](https://github.com/n3uz/ELKR-STACK/blob/master/ES%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE.md)   

对外统一提供Logstash监听的8001端口，用于接收各个Agent收集上来的日志。多加个logstash在前面监听8001，是把Redis保护起来，你设置个密码访问Redis吧，又得在收集日志的客户端配置上redis的访问密码，容易泄漏。Redis负责中转，再交给Logstash做正则匹配，最后存储到ES集群，Kibana做分析统计展现。Nginx反向代理Kibana。

###  三、测试环境方案     
硬件配置：   

- CentOS7     
- CPU:8C  
- RAM:16G     
- DISK:300G   

因为网络环境复杂，下载ELK介质的速度又不行，我把个人收集的介质上传，请对比HASH合理食用。	
我建议如果是想做实验，有美利坚的VPS 1000M网络共享，下载速度感动到你哭。	
而且也支持在线安装各种插件。这个你在大TC想都不敢想。

### 四、实施方案        
1. 检查介质清单    

  - CentOS Linux release 7.1.1503 (Core)或其他    
  - jdk1.8    
  - Elasticsearch5.3.2    
  - Logstash5.3.2 
  - Kibana5.3.2   
  - Filebeat5.3.2 
  - Redis3.2.6    
  - Nginx1.10 yum安装     
所有介质打包下载 [百度网盘地址]()    TODO

创建目录elkr，上传所有包到/elkr     

```bash
mkdir -p /elkr/ 
```
    
2. 部署工作环境，调优系统参数       

```bash
yum -y install gcc tcl
```

设置java环境变量

```bash
tar zxvf jdk.tar.gz
vi /etc/profile
```

```bash
  export JAVA_HOME=/elkr/jdk18
  export CLASSPATH=$JAVA_HOME/libs/dt.jar:$JAVA_HOME/tools.jar
  export PATH=$PATH:$JAVA_HOME/bin/
```

确保java工作正常

```bash
java -verison
```

调整内核参数

```bash
vi /etc/security/limits.conf
```

```
  * soft nofile 65536
  * hard nofile 65536
  * soft process 65536
  * hard process 65536
  * soft memlock 65536
  * hard memlock 65536
```

```bash
echo 'vm.max_map_count = 262144' >> /etc/sysctl.conf
echo 'vm.overcommit_memory = 1' >> /etc/sysctl.conf
sysctl -p
```


3. 安装redis3.2.6版本为例   

```bash
tar zxvf redis3.2.6.tar.gz
mv redis3.2.6 redis
make MALLOC=libc v=1
make test
make install
```

```bash
vi /etc/rc.local
```

```bash
echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
```


编辑配置文件

```bash
vi /etc/redis.conf
```

```bash
  bind 127.0.0.1 #因为不需要对外提供服务，所以监听本地即可，方便后面不设置密码访问
  protected-mode no
  port 6379
  tcp-backlog 511
  timeout 0
  tcp-keepalive 300
  daemonize yes
  supervised no
  pidfile /var/run/redis_6379.pid
  loglevel notice
  logfile "/elkr/redis/redis.log" #log路径目录，需要提前创建
  databases 16
  save 900 1
  save 300 10
  save 60 10000
  stop-writes-on-bgsave-error yes
  rdbcompression yes
  rdbchecksum yes
  dbfilename dump.rdb
  dir ./
  slave-serve-stale-data yes
  slave-read-only yes
  repl-diskless-sync no
  repl-diskless-sync-delay 5
  repl-disable-tcp-nodelay no
  slave-priority 100
  requirepass Passw0rd
  appendonly no
  appendfilename "appendonly.aof"
  appendfsync everysec
  no-appendfsync-on-rewrite no
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  aof-load-truncated yes
  lua-time-limit 5000
  slowlog-log-slower-than 10000
  slowlog-max-len 128
  latency-monitor-threshold 0
  notify-keyspace-events ""
  hash-max-ziplist-entries 512
  hash-max-ziplist-value 64
  list-max-ziplist-size -2
  list-compress-depth 0
  set-max-intset-entries 512
  zset-max-ziplist-entries 128
  zset-max-ziplist-value 64
  hll-sparse-max-bytes 3000
  activerehashing yes
  client-output-buffer-limit normal 0 0 0
  client-output-buffer-limit slave 256mb 64mb 60
  client-output-buffer-limit pubsub 32mb 8mb 60
  hz 10
  aof-rewrite-incremental-fsync yes
```

启动redis

```bash
redis-server /etc/redis.conf
```

验证redis是否可用

```bash
redis-cli 
127.0.0.1:6379> keys *
```

验证Redis安装配置成功

4. 部署ES   
单机部署方案 | [集群部署方案](https://github.com/n3uz/ELKR-STACK/blob/master/ES%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE.md)

```bash
unzip elasticsearch-5.3.2.zip
mv elasticsearch-5.3.2 es5.3.2
cd es5.3.2
vi config/elasticsearch.yml
```

```bash
    cluster.name: Your_cluster_name
    node.name: Node-1
    path.data: /elkr/es5.3.2/data
    path.logs: /elkr/es5.3.2/logs
```

修改jvm参数，根据实际情况配置，建议两个一样。其他参数保持默认

```bash
vi jvm.options
```

```
-Xms4g
-Xmx4g
```

启动ES

```bash
/elkr/es5.3.2/bin/elasticsearch -d
```

验证ES工作正常

```
[root@elk config]# curl -XGET "http://localhost:9200"
{
  "name" : "node-1",
  "cluster_name" : "CA-ES-STACK",
  "cluster_uuid" : "6AXF-GcjROKB43RQ0F9XZg",
  "version" : {
    "number" : "5.3.2",
    "build_hash" : "3068195",
    "build_date" : "2017-04-24T16:15:59.481Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.2"
  },
  "tagline" : "You Know, for Search"
}

```


5. 部署Kibana   

```bash
tar zxvf kibana-5.3.2-linux-x86_64.tar.gz
mv kibana-5.3.2-linux-x86_64 kibana
cd kibana
vim config/kibana.yml
```

```bash
    server.port: 5601
    server.host: "localhost"
    elasticsearch.url: "http://localhost:9200"
    kibana.index: ".kibana"
```


只监听本地的5601端口，后面会使用nginx反向代理，提供其他主机访问。   

启动kibana

```bash
/elkr/kibana/bin/kibana
```

启动kibana脚本

```bash
vi start-kibana.sh
```

```bash
nohup /elkr/kibana/bin/kibana </dev/null &>/dev/null &
```
检查kibana启动是否成功

```bash
netstat -an|grep 5601
```


6. 安装nginx配置代理
在线yum安装nginx

```bash
vim /etc/yum.repo/nginx.repo
```

```bash
  [nginx]
  name=nginx repo
  baseurl=http://nginx.org/packages/centos/7/$basearch/
  gpgcheck=0
  enabled=1
```

```bash
yum -y install nginx
```


使用nginx代理与配置密码验证 


```bash
vi /etc/nginx/conf.d/default.conf
```

```bash
server { 
listen 80; 
    server_name localhost; 
    location / { 
        auth_basic "secret"; 
        auth_basic_user_file /etc/nginx/conf.d/passwd.db; #由下面的工具在线生成
        proxy_pass http://127.0.0.1:5601;
        proxy_set_header Host $host:5601; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_set_header Via "nginx"; 
    } 
    access_log /var/log/nginx/access.log main;
}
```


htpasswd 在线生成 ,[戳我传送](http://tool.oschina.net/htpasswd)  
检查配置，启动nginx     

```bash
nginx -t
nginx
```

访问http://IP，输入上面设置的htpasswd的用户名密码登录，测试正常。   

7. 部署Logstash 监听8001端口，用于接收日志，并转发到redis

```bash
tar zxvf logstash-5.3.2.tar.gz
mv logstash-5.3.2 logstash
cd logstash
```


```
vim config/conf-8001.conf
```


```
input {
	beats {
		port => 8001
	}
}
output {
if [type]=="nginx:access" { 
#type区分开不同主机的不同日志
#type的设置将会从filebeat处增加一个字段
      redis {
		host => "127.0.0.1"
		key => "nginx:access" #不同来源的数据存放到不同list里
		port => 6379
		data_type => "list"
      }
    }
}
```


启动logstash    

```bash
nohup /elkr/ls5.3.2/bin/logstash -f /elkr/ls5.3.2/config/filebeat.conf -l /elkr/ls5.3.2/log/filebeat/ &
```

查看启动日志与端口，确认启动正常。  

```
[root@elk config]# ss -ltn|grep 8001
LISTEN     0      128  *:8001                     *:*
```
                 

8. 在客户端部署filebeat，发送文件日志到logstash8001  

为了避免filebeat无权限读取日志文件，请为相应的日志文件赋予可读权限      

```
su - root
tar zxvf filebeat-5.3.2-linux-x86_64.tar.gz
mv filebeat-5.3.2 filebeat
vim filebeat.yml
```

```
filebeat.prospectors:
- input_type: "log"
  document_type: "nginx:access"
  paths:
    - /var/log/nginx/access.log
    #- c:\programdata\elasticsearch\logs\*

output.logstash:
  hosts: ["127.0.0.1:8001"]
```

以上配置：读取本机的/var/log/nginx/access.log，发送到logstash 127.0.0.1:8001，  
且标记为document_type: "nginx:access"，这样便于存放到redis的不同list中。 

启动filebeat    

```
/elkr/filebeat/filebeat -e -c /elkr/filebeat/filebeat.yml &
```

连接到redis服务器，确认redis工作正常    

```
[root@elk config]# redis-cli 
127.0.0.1:6379> keys *
(1) nginx:access
```

标明日志文件成功通过logstash8001端口存储到redis  

9. 部署Logstash完成正则匹配，切分日志域    


建立用来存放Grok正则表达式文件的路径    

```
mkdir -p /elkr/ls5.3.2/patterns
```

将grok_patterns文件，与自定义pattern文件存放在此目录  

```
vim /elkr/ls5.3.2/patterns/grok_patterns
```

预定义表达式见：[grok_patterns](https://grokdebug.herokuapp.com/patterns)       

以下为自定义nginx access日志正则匹配  
```
vim /elkr/ls5.3.2/patterns/nginx-access
```


```
YourHostIP-N-ACCESS %{IPORHOST:remote_addr} - (%{USERNAME:remote_user}|-) \[%{HTTPDATE:timestamp}\] "%{GREEDYDATA:request}"%{NUMBER:CODE} %{NUMBER:SIZE} "%{DATA:http_referer}" %{HOST:http_host} %{URIPATHPARAM:request_uri} (?:%{QUOTEDSTRING:UA}|-) "-" (?:%{HOSTPORT1:upstream_addr}|-) - %{NUMBER:upstream_code} %{NUMBER:upstream_response}
HOSTPORT1 (%{IPV4}:%{POSINT}[, ]{0,2})+
```


YourHostIP-N-ACCESS 与 HOSTPORT1为自定义表达式。	
为了能够编写匹配不同格式的nginx日志，需要定义不同的名字，所以采用YourHostIP-N-ACCESS这样的命名，后面会用到。	
HOSTPORT1表达式是为了匹配出 IP: PORT 这类型的数据，仅仅是个人需求。    
一个正则表达式在线调试的网站GrokDebugger，国内访问可能会慢。	
[Grok Debugger](http://grokdebug.herokuapp.com/)
```
vim  /elkr/ls5.3.2/config/redis2es.conf
```


```
input {
	 redis {
                host => "127.0.0.1"
      		key => "nginx:access"
          	port => 6379
          	data_type => "list"
          	
          }

}
filter {
  if [type] == "nginx:access" {
    grok {
	    patterns_dir => "/elkr/ls5.3.2/patterns"
	    match => [ 
	        "message","%{YourHostIP-N-ACCESS}"]
    }

    if "_grokparsefailure" in [tags] {
          drop {} 
    }
    
    mutate{
      remove_field => [ "message" ]
      remove_field => [ "path" ]
      remove_field => ["hostname"]
      
    }
  }
}

output {
    if [type] == "nginx:access" {
        elasticsearch {
                hosts => ["127.0.0.1"]
                index => "nginx-access-%{+YYYY.MM.dd}"
        }
    }
}
```

配置文件一共分三段：

- 从Redis输入
- Grok匹配过滤
- 输出到ES

具体语法请参照Logstash官方手册      

10. Kibana展现  

访问http://IP   输入上面设置的htpasswd的用户名密码登录。  

```
http://localhost
```


【可选】为了个人需求，我修改了一下模版      

点击左边菜单开发工具，运行如下代码，将所有字段与他的raw数据都保留。     

```
PUT _template/all-string-with-raw
 {
    "order": 0,
    "template": "*",
    "settings": {
      "index": {
        "refresh_interval": "5s"
      }
    },
    "mappings": {
      "_default_": {
        "dynamic_templates": [
          {
            "message_field": {
              "mapping": {
                "fielddata": {
                  "format": "disabled"
                },
                "index": "analyzed",
                "omit_norms": true,
                "type": "string"
              },
              "match_mapping_type": "string",
              "match": "message"
            }
          },
          {
            "string_fields": {
              "mapping": {
                "fielddata": {
                  "format": "disabled"
                },
                "index": "analyzed",
                "omit_norms": true,
                "type": "string",
                "fields": {
                  "raw": {
                    "ignore_above": 256,
                    "index": "not_analyzed",
                    "type": "string"
                  }
                }
              },
              "match_mapping_type": "string",
              "match": "*"
            }
          }
        ],
        "properties": {
          "@timestamp": {
            "type": "date"
          },
          "geoip": {
            "dynamic": true,
            "properties": {
              "latitude": {
                "type": "float"
              },
              "ip": {
                "type": "ip"
              },
              "location": {
                "type": "geo_point"
              },
              "longitude": {
                "type": "float"
              }
            }
          },
          "@version": {
            "index": "not_analyzed",
            "type": "string"
          }
        },
        "_all": {
          "omit_norms": true,
          "enabled": true
        }
      }
    },
    "aliases": {}
  }
```

创建indices

  点击左边菜单Management->Index Patterns-> Configure an index pattern-> 	
Index name or pattern处输入第9步配置文件    

```
index => "nginx-access-%{+YYYY.MM.dd}"
```
中的index名，如我的例子中就应该是nginx-access-\*，因为索引会按天创建，所以请保留末尾的*。   

创建Visualize、Dashboard

  左边菜单Discover为定义的Index Pattern的展现，也是后面Visualize的数据基础，	
Visualize做出的图表，通过Dashboard展现。这样就完成了Kibana的视图设置，Kibana支持多种格式统计图表，可以深入挖掘使用。 


至此ELKR STACK搭建完毕。    
其中耗时的地方在于正则表达式匹配日志。  
恰好性能瓶颈也很大程度上会出在这里，可以考虑加入行处理的工具来规避大数据量下的Grok效率低的问题。    


###  五、参考手册      
[Elastic.co文档](https://www.elastic.co/guide/index.html)
###  六、踩过的坑       
慢慢总结、细细品尝

1. 强制掉电/ES索引坏了

那天我脑袋抽了，直接reboot了

系统起来以后启动ES发现有问题了。
```
log   [14:27:08.487] [error][status][plugin:elasticsearch@1.0.0] Status changed from red to red - Elasticsearch is still initializing the kibana index.
```

而且等了很久一直不能用，Google了一下，参照这个[问题](http://stackoverflow.com/questions/31201051/elasticsearch-is-still-initializing-the-kibana-index)，里面提到了两种解决方案。

A. 解决方案一，不用删除索引
```
curl -s http://localhost:9200/.kibana/_recovery?pretty
curl -XPUT 'localhost:9200/.kibana/_settings' -d '
{
    "index" : {
        "number_of_replicas" : 0
    }
}'
```

亲测无效

B. 删除kibana索引，这样所有建立的visualize和Dashboard都没有了，甚至要删除所有索引

```
curl -XDELETE http://localhost:9200/.kibana
curl -XDELETE http://localhost:9200/*
```

那么你猜我索引有没有删除 :hear_no_evil:

2. logstash grok并没有你想的那么高效

在处理单条日志内容过大时，我的大概2k，grok的效率简直不能忍，新手又没有其他姿势来切分匹配。朋友建议说使用其他的行处理工具，这里还要埋个坑

3. 待发现


</font>

