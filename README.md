
<font face="微软雅黑"> 

## 《ELKR STACK从入门到放弃》   
ELKR Stack 日志监控平台     
 ![假装有图片](https://)


##### 1. ES 单节点测试环境 ALL IN ONE

[ES 单节点测试环境部署方案](https://github.com/n3uz/ELKR-STACK/blob/master/ES%E5%8D%95%E8%8A%82%E7%82%B9%E6%96%B9%E6%A1%88%E6%B5%8B%E8%AF%95%E6%96%B9%E6%A1%88ALL%20IN%20ONE)

![image](https://github.com/n3uz/ELKR-STACK/blob/master/ELKR%E6%9E%B6%E6%9E%84%E5%9B%BE%EF%BC%88ES%E5%8D%95%E8%8A%82%E7%82%B9%EF%BC%89.png) 


#### 2. ES 生产环境集群模式

[ES Cluster 生产环境部署方案](https://github.com/n3uz/ELKR-STACK/blob/master/ES%E9%9B%86%E7%BE%A4%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E6%96%B9%E6%A1%88)

![ELKR ES集群架构](https://github.com/n3uz/ELKR-STACK/blob/master/ELKR%E6%9E%B6%E6%9E%84%E5%9B%BE%EF%BC%88ES%E9%9B%86%E7%BE%A4%EF%BC%89.png)

#### 3. 这个问题必须拿在这里来说

```
output {
    if [type] == "nginx:access" {
        elasticsearch {
                hosts => ["127.0.0.1"]
                index => "nginx-access-%{+YYYY.MM.dd}"
        }
    }
```
在网上找到的资料很多都这么写的，index => "nginx-access-%{+YYYY.MM.dd}"，为每天生成一个indice，但是一旦拥有了几千个indices的时候，重启ES，一直提示在恢复索引。你会看到提示：
```
{"name":"Kibana","hostname":"ABCD","pid":3848,"level":30,"msg":"Elasticsearch is still initializing the kibana index... Trying again in 2.5 second.","time":"2017-05-03T07:35:34.936Z","v":0}

```
目前没有找到更好的解决方案，网上找到的资料都是:  **删除所有indices** 
```
curl -XDELETE http://localhost:9200/*
```
我想没有几个人想这么做。

所以我根据日志的量，有些indice按年，有些按月，我想我不会再按日了。

```
index => "nginx-access-%{+YYYY}"
index => "nginx-access-%{+YYYY.MM}"
```

</font>
