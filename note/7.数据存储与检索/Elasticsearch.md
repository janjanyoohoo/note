



## 集群搭建

### 新建节点文件夹

```c
mkdir -p xxx
# 目录结构  
es
|---/config
|	 |---node1-config.yml
|    |---node2-config.yml
|    |---node3-config.yml
|---/logs
|	 |---node1-log.txt
|	 |---node2-log.txt
|	 |---node3-log.txt
|---/data
	 |---/node1-data
	 |---/node2-data
	 |---/node3-data
```

### 写入配置文件

```c
uster.name: janjan
# 设置节点名称，集群内节点名称必须唯一。
node.name: es-node1
# 表示该节点会不会作为主节点，true表示会；false表示不会
node.master: true
# 当前节点是否用于存储数据，是：true、否：false
node.data: true
# 索引数据存放的位置
path.data: /user-file/es/data-node1
# 日志文件存放的位置
path.logs: /user-file/es/logs
# 需求锁住物理内存，是：true、否：false
#bootstrap.memory_lock: true
# 监听地址，用于访问该es
network.host: 0.0.0.0
# es对外提供的http端口，默认 9200
http.port: 9200
# TCP的默认监听端口，默认 9300
transport.tcp.port: 9300
# 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）
discovery.zen.minimum_master_nodes: 1
# es7.x 之后新增的配置，写入候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["172.17.63.252:9300", "172.17.63.252:9301","172.17.63.252:9302"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5
# es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["172.17.63.252:9300", "172.17.63.252:9301","172.17.63.252:9302"]
# 是否支持跨域，是：true，在使用head插件时需要此配置
http.cors.enabled: true
# “*” 示支持所有域名
http.cors.allow-origin: "*"

    
    #集群名称
cluster.name: my-es
#当前该节点的名称
node.name: node-1
#是不是有资格竞选主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 3
#给当前节点自定义属性（可以省略）	
#node.attr.rack: r1
#数据存档位置
path.data: /usr/share/elasticsearch/data
#日志存放位置
path.logs: /usr/share/elasticsearch/log
#是否开启时锁定内存（默认为是）
#bootstrap.memory_lock: true
#设置网关地址，我是被这个坑死了，这个地址我原先填写了自己的实际物理IP地址，
#然后启动一直报无效的IP地址，无法注入9300端口，这里只需要填写0.0.0.0
network.host: 0.0.0.0
#设置其它结点和该结点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址，设置当前物理机地址,
#如果是docker安装节点的IP将会是配置的IP而不是docker网管ip
network.publish_host: 192.168.114.136
#设置映射端口
http.port: 9200
#内部节点之间沟通端口
transport.tcp.port: 9300
#集群发现默认值为127.0.0.1:9300,如果要在其他主机上形成包含节点的群集,如果搭建集群则需要填写
#es7.x 之后新增的配置，写入候选主节点的设备地址，在开启服务后可以被选为主节点，也就是说把所有的节点都写上
discovery.seed_hosts: ["192.168.114.136:9300","192.168.114.136:9301","192.168.114.136:9302"]
#当你在搭建集群的时候，选出合格的节点集群，有些人说的太官方了，
#其实就是，让你选择比较好的几个节点，在你节点启动时，在这些节点中选一个做领导者，
#如果你不设置呢，elasticsearch就会自己选举，这里我们把三个节点都写上
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
#在群集完全重新启动后阻止初始恢复，直到启动N个节点
#简单点说在集群启动后，至少复活多少个节点以上，那么这个服务才可以被使用，否则不可以被使用，
gateway.recover_after_nodes: 2
#删除索引是是否需要显示其名称，默认为显示
#action.destructive_requires_name: true

cluster.name: es-cluster
node.name: node-3
node.master: true
node.data: true
node.max_local_storage_nodes: 3
path.data: /usr/share/elasticsearch/data
path.logs: /usr/share/elasticsearch/log
network.host: 0.0.0.0
network.publish_host: 10.0.8.16
http.port: 9202
transport.tcp.port: 9302
discovery.seed_hosts: ["10.0.8.16:9300","10.0.8.16:9301","10.0.8.16:9302"]
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
gateway.recover_after_nodes: 1
#action.destructive_requires_name: true


docker run -e ES_JAVA_OPTS="-Xms512m -Xms512m" --network es-net  -d -p 9200:9200 -p 9300:9300 -v /home/elasticsearch/es-node1/config/es-config.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/elasticsearch/es-node1/plugins:/usr/share/elasticsearch/plugins -v /home/elasticsearch/es-node1/data:/usr/share/elasticsearch/data -v /home/elasticsearch/es-node1/log:/usr/share/elasticsearch/log --name es-node-1 elasticsearch:7.6.0
    
docker run -e ES_JAVA_OPTS="-Xms128m -Xms128m" --network es-net -d -p 9201:9201 -p 9301:9301 -v /home/elasticsearch/es-node2/config/es-config.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/elasticsearch/es-node2/plugins:/usr/share/elasticsearch/plugins -v /home/elasticsearch/es-node2/data:/usr/share/elasticsearch/data -v /home/elasticsearch/es-node2/log:/usr/share/elasticsearch/log --name es-node-2 elasticsearch:7.6.0
        
docker run -e ES_JAVA_OPTS="-Xms128m -Xms128m" --network es-net -d -p 9202:9202 -p 9302:9302 -v /home/elasticsearch/es-node3/config/es-config.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/elasticsearch/es-node3/plugins:/usr/share/elasticsearch/plugins -v /home/elasticsearch/es-node3/data:/usr/share/elasticsearch/data -v /home/elasticsearch/es-node3/log:/usr/share/elasticsearch/log --name es-node-3 elasticsearch:7.6.0

    


        
```

