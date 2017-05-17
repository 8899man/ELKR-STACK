#####  ES 集群生产环境方案

![image](https://github.com/n3uz/ELKR-STACK/blob/master/ELKR%E6%9E%B6%E6%9E%84%E5%9B%BE%EF%BC%88ES%E9%9B%86%E7%BE%A4%EF%BC%89.png) 

#### 1~3 请参照单机版部署

####  4. 部署ES 集群  

以三个节点为例

- 1.1.1.2 ES  Cilent mode ,KIBANA NGINX
- 1.1.1.3 ES  master mode ,data mode
- 1.1.1.4 ES  master mode ,data mode

登录1.1.1.2 ，解压缩介质
```bash
unzip elasticsearch-5.3.2.zip
mv elasticsearch-5.3.2 es5.3.2
cd es5.3.2
vi config/elasticsearch.yml
```

```bash
cluster.name: CA-ES-Cluster
node.name: node-1.1.1.2
node.master: false
node.data: false
path.data: /data/es_cluster/data
path.logs: /data/es_cluster/logs
network.host: 127.0.0.1,1.1.1.2
http.port: 9200
discovery.zen.ping.unicast.hosts: ["1.1.1.2:9300","1.1.1.3:9300", "1.1.1.4:9300"]
discovery.zen.minimum_master_nodes: 2
gateway.recover_after_nodes: 2
```

修改jvm参数，根据实际情况配置，建议两个一样。其他参数保持默认

```bash
vi jvm.options
```

```
-Xms4g
-Xmx4g
```

同理，准备其他两台node的介质与配置文件。如下

- 1.1.1.3 master mode, data mode

```
cluster.name: CA-ES-Cluster
node.name: node-1.1.1.3
node.master: true
node.data: true
path.data: /data/es_cluster/data
path.logs: /data/es_cluster/logs
network.host: 127.0.0.1,1.1.1.3
http.port: 9200
discovery.zen.ping.unicast.hosts: ["1.1.1.4:9300"]

```
- 1.1.1.4 master mode, data mode

```
cluster.name: CA-ES-Cluster
node.name: node-1.1.1.4
node.master: true
node.data: true
path.data: /data/es_cluster/data
path.logs: /data/es_cluster/logs
network.host: 127.0.0.1,1.1.1.4
http.port: 9200
discovery.zen.ping.unicast.hosts: ["1.1.1.3:9300"]
```

>确保1.1.1.2，1.1.1.3，1.1.1.4之间9300能够互相访问，开通相应防火墙规则。

启动ES，依次在三台服务器上执行

```bash
/elkr/es5.3.2/bin/elasticsearch -d
```

验证ES工作正常

```
[root@localhost ~]# curl -XGET "1.1.1.4:9200/_cat/nodes?pretty"
1.1.1.3 10 87  1 0.02 0.03 0.05 mdi * node-1.1.1.3
1.1.1.4  9 99  1 0.00 0.01 0.05 mdi - node-1.1.1.4
1.1.1.2  6 99 47 1.44 0.60 0.28 i   - node-1.1.1.2

```
node-1.1.1.2 工作在Client模式（i），其余两个工作在MASTER与DATA模式（mdi），当前集群MASTER是 node-1.1.1.3


####  5. 部署Kibana   

在1.1.1.2上部署kibana，使用如下配置
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
至此，ES集群方案完毕

#### 6. 安装nginx配置代理

参考单机方案   

#### 7. 部署Logstash 监听8001端口，用于接收日志，并转发到redis

参考单机方案                 

#### 8. 在客户端部署filebeat，发送文件日志到logstash8001  

参考单机方案 

#### 9. 部署Logstash完成正则匹配，切分日志域    

参考单机方案
>这一步跟单机方案差不多，只是在Logstash输出到ES部分，需要配置两个ES data node的IP地址与端口。做到ES集群的Load Balance


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
                hosts => ["1.1.1.3:9200","1.1.1.4:9200"]
                index => "nginx-access-%{+YYYY.MM.dd}"
        }
    }
}
```
      

#### 10. Kibana展现  

参照单机方案

至此ELKR ES集群方案完毕

###  * 参考手册      
[Elastic.co文档](https://www.elastic.co/guide/index.html)
