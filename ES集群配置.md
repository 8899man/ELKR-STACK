### ES 集群配置

下面以三个节点为例，配置ES集群    

- 1.1.1.2 ES  Cilent mode ,KIBANA #工作在Client mode，负载均衡地址
- 1.1.1.3 ES  master mode ,data mode,
- 1.1.1.4 ES  master mode ,data mode

##### 1. 配置1.1.1.2工作在client mode

登录1.1.1.2，编辑 elasticsearch.yml 文件

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

##### 2. 配置1.1.1.3工作在master mode ,data mode

登录1.1.1.3，编辑 elasticsearch.yml 文件
```bash
cluster.name: CA-ES-Cluster
node.name: node-1.1.1.3
node.master: true
node.data: true
path.data: /data/es_cluster/data
path.logs: /data/es_cluster/logs
bootstrap.mlockall: true
network.host: 127.0.0.1,1.1.1.3
http.port: 9200
discovery.zen.ping.unicast.hosts: ["1.1.1.2:9300","1.1.1.3:9300", "1.1.1.4:9300"]
discovery.zen.minimum_master_nodes: 2
gateway.recover_after_nodes: 2
```


##### 3. 配置1.1.1.4工作在master mode ,data mode

登录1.1.1.4，编辑 elasticsearch.yml 文件

```bash
cluster.name: CA-ES-Cluster
node.name: node-1.1.1.4
node.master: true
node.data: true
path.data: /data/es_cluster/data
path.logs: /data/es_cluster/logs
bootstrap.mlockall: true
network.host: 127.0.0.1,1.1.1.4
http.port: 9200
discovery.zen.ping.unicast.hosts: ["1.1.1.2:9300","1.1.1.3:9300", "1.1.1.4:9300"]
discovery.zen.minimum_master_nodes: 2
gateway.recover_after_nodes: 2
```

在三台主机上分别启动ES
```
ES_HOME/bin/elasticsearch -d
```

查看集群日志，确认启动正常，且日志中会有加入集群与选举master的日志

```
view /data/es_cluster/logs/CA-ES-Cluster.log
```

```
[2017-05-17T00:35:20,179][INFO ][o.e.n.Node               ] [node-1.1.1.2] initializing ...
[2017-05-17T00:35:20,276][INFO ][o.e.e.NodeEnvironment    ] [node-1.1.1.2] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [25.1gb], net total_space [26.9gb], spins? [unknown], types [rootfs]
[2017-05-17T00:35:20,277][INFO ][o.e.e.NodeEnvironment    ] [node-1.1.1.2] heap size [1015.6mb], compressed ordinary object pointers [true]
[2017-05-17T00:35:20,278][INFO ][o.e.n.Node               ] [node-1.1.1.2] node name [node-1.1.1.2], node ID [oZzke4ioQU2G7RS8Idp98g]
[2017-05-17T00:35:20,278][INFO ][o.e.n.Node               ] [node-1.1.1.2] version[5.3.2], pid[1677], build[3068195/2017-04-24T16:15:59.481Z], OS[Linux/3.10.0-514.el7.x86_64/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_45/25.45-b02]
[2017-05-17T00:35:23,108][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [aggs-matrix-stats]
[2017-05-17T00:35:23,108][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [ingest-common]
[2017-05-17T00:35:23,108][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [lang-expression]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [lang-groovy]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [lang-mustache]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [lang-painless]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [percolator]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [reindex]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [transport-netty3]
[2017-05-17T00:35:23,109][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] loaded module [transport-netty4]
[2017-05-17T00:35:23,110][INFO ][o.e.p.PluginsService     ] [node-1.1.1.2] no plugins loaded
[2017-05-17T00:35:30,756][INFO ][o.e.n.Node               ] [node-1.1.1.2] initialized
[2017-05-17T00:35:30,779][INFO ][o.e.n.Node               ] [node-1.1.1.2] starting ...
[2017-05-17T00:35:31,229][INFO ][o.e.t.TransportService   ] [node-1.1.1.2] publish_address {1.1.1.2:9300}, bound_addresses {127.0.0.1:9300}, {1.1.1.2:9300}
[2017-05-17T00:35:31,246][INFO ][o.e.b.BootstrapChecks    ] [node-1.1.1.2] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks

[2017-05-17T00:35:34,614][INFO ][o.e.c.s.ClusterService   ] [node-1.1.1.2] detected_master {node-1.1.1.3}{H8QYFJ7SQBefSXXLJLbhbQ}{Wv2OYycyT_qKHetID6cvRQ}{1.1.1.3}{1.1.1.3:9300}, added {{node-1.1.1.3}{H8QYFJ7SQBefSXXLJLbhbQ}{Wv2OYycyT_qKHetID6cvRQ}{1.1.1.3}{1.1.1.3:9300},{node-1.1.1.4}{hcG4yr5KQHaeCDhhaROtGA}{l2Xil8osTyO7EGpZDW9fzQ}{1.1.1.4}{1.1.1.4:9300},}, reason: zen-disco-receive(from master [master {node-1.1.1.3}{H8QYFJ7SQBefSXXLJLbhbQ}{Wv2OYycyT_qKHetID6cvRQ}{1.1.1.3}{1.1.1.3:9300} committed version [119]])

[2017-05-17T00:35:34,725][INFO ][o.e.h.n.Netty4HttpServerTransport] [node-1.1.1.2] publish_address {1.1.1.2:9200}, bound_addresses {127.0.0.1:9200}, {1.1.1.2:9200}
[2017-05-17T00:35:34,737][INFO ][o.e.n.Node               ] [node-1.1.1.2] started
[2017-05-17T00:44:04,169][INFO ][o.e.m.j.JvmGcMonitorService] [node-1.1.1.2] [gc][511] overhead, spent [530ms] collecting in the last [1.4s]

```

##### 4. 通过API查看集群状态

```bash
curl -XGET "1.1.1.2:9200/_cat/nodes?pretty"
```

```
1.1.1.3 9 99 0 0.00 0.01 0.05 mdi * node-1.1.1.3
1.1.1.2 8 99 5 1.05 0.66 0.34 i   - node-1.1.1.2
1.1.1.4 8 99 0 0.08 0.06 0.07 mdi - node-1.1.1.4
```

node-1.1.1.2 工作在Client模式（i），其余两个工作在MASTER与DATA模式（mdi），当前集群MASTER是 node-1.1.1.3

#### 至此，ES集群配置部分完毕。


##### TODO 2017-05-17

ES的9200端口对外开放，没有设定有效的访问控制。
计划通过iptables保护 ，[戳这里传送](https://github.com/n3uz/ELKR-STACK/blob/master/%E4%BF%9D%E6%8A%A4ES%E7%9A%84iptables%E7%AD%96%E7%95%A5.md)
