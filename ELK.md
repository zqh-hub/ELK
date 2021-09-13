#### ELK

##### ELK介绍

```
ELK是Elasticsearch、Logstash、Kibana三个软件
Elasticsearch：全文搜索
Logstash：日志收集工具，可以实现日志内容收集及各式转换
Kibana：页面管理工具，可以通过它的洁面操作Elasticsearch
```

##### Elasticsearch相关概念

###### 1、Cluster

```
集群。Elasticsearch集群由一或多个节点组成，其中有一个主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。Elasticsearch的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部看Elasticsearch集群，在逻辑上是个整体，你与集群中的任何一个节点通信和与整个Elasticsearch集群通信是等价的。也就是说，主节点的存在不会产生单点安全隐患、并发访问瓶颈等问题
```

###### 2、Index

```python
索引，相当于关系数据库中的表。其中存储若干相似结构的Docment数据。如：客户索引，订单索引等。Elasticsearch中的索引不像数据库表一样有强制的数据结构约束，在理论上，可以存储任意结构的数据。但为了更好的为业务提供数据支撑，还是要设计合适的索引体系来存储不同的数据

数据库：库--->表--->字段
Elasticsearch：索引(Index)--->文档(Docment)--->字段(field)
```

###### 3、Shards

```
primary shards:代表主分片，Elasticsearch可以把一个完整的索引分成多个primary shards，这样的好处是可以把一个大的索引拆分成多个分片，分布存储在不同的Elasticsearch节点上，从尔形成分布式存储，并为搜索访问提供分布式服务，提高并发处理能力。primary shard的数量只能在索引创建时指定，并且索引创建后不能再更换primary shard数量
```

###### 4、Replicas

```
replicas shard：代表主分片的副本分片，Elasticsearch可以设置多个replica shard。它的作用：一是提高系统的容错性，当某个节点某个primary shard损坏或丢失时，可以从副本中恢复。二是提高Elasticaearch的查询效率，Elasticaearch会自动对搜索请求进行负载均衡，将并发的搜索请求发送给合适的节点，增强并发处理能力
```

![01](/Users/coco/Documents/Learn/ELK/img/01.png)

###### 5、Docment

```
文档。Elasticsearch中最小的数据单位。一个Docment就是一条数据，一般使用json数据结构。每个Index下的T ype只能够都可以存储多个Docment。一个Docment中可定义多个field,field就是数据字段

Elasticsearch对Docment中各个字段没有限制，也就是说Docment之间的字段可以不同
```

###### 6、元数据

```
在Elasticsearch中多有以“_”开头的属性都称为元数据，都有着自己特定的含义。如：_index,相当于key-value的ke y
```

###### 7、倒排索引(Elasticsearch索引原理)

```
对于数据进行分析，抽取出数据中的词条，以词条作为key，对应数据的存储位置作为value，实现索引的存储。这种索引被称为倒排索引。倒排索引是Document写入Elasticsearch是分析维护的。
```

![02](/Users/coco/Documents/Learn/ELK/img/02.png)

##### ELK安装(Centos 7)

###### 1、修改文件个数限制

```shell
# linux最多创建的文件是65535个，但是Elasticsearch至少需要65536的文件创建权限
vim /etc/security/limits.conf
* soft nofile 65536  * 任何用户，soft 内存中，nofile 文件权限
* hard nofile 65536  hard 硬盘中
```

###### 2、修改线程开启限制

```shell
修改系统中允许用户启动的进程开启多少个线程。默认Linux限制root用户开启的进程可以开始任意数量的线程，其他用户开启的进程可以开启1024个线程。必须修改限制数为4096+。因为Elasticsearch至少需要4096的线程池预备。Elasticsearch在5.x版本后，强制要求在linux中不能使用root用户启动Elasticsearch进程，所以必须使用其他用户启动
vim /etc/srcurity/limits.conf
* soft nproc 4096     # 任何用户的创建线程数最高达4096
root soft nproc unlimited  # root没限制

# 虚拟机内存是1G,最多能开启3000+个线程，所以最少1.5G
```

###### 3、修改系统的控制文件

```
系统控制文件是管理系统中的各种资源控制的配置文件。Elasticsearch需要开辟一个65536字节以上的虚拟内存空间。Linux默认不允许任何用户和应用直接开辟虚拟内存

vim /etc/sysctl.d/99-sysctl.conf
vm.max_map_count=655360

修改后执行：sysctl -p
```

###### 4、Docker安装

```shell
docker pull elasticsearch:7.6.2

# 启动
[root@172 docker]# docker run --name=es -d -p 9200:9200 -p 9300:9300 --restart=always -e "discovery.type=single-node" elasticsearch:7.6.2

7.0版本之前，9300端口是java代码访问的端口，9200端口是http访问的
7.0版本之后，都用9200端口访问，所以这里不开启9300也可以

# 验证
[root@172 docker]# curl http://localhost:9200
```

###### 5、Kibana安装

```shell
# kibana要与elasticsearch版本一模一样
docker pull kibana:7.6.2

# 启动kibana
docker run -it -d --name kibana --link es:es -p 5601:5601 kibana:7.6.2

# 修改kibana参数
[root@172 docker]# docker exec -it kibana /bin/bash
bash-4.2$ cd config/
apm.js	kibana.yml
bash-4.2$ vi kibana.yml 
elasticsearch.hosts: [ "http://172.16.39.2:9200" ]  # 改成自己虚拟机的ip

bash-4.2$ exit
# 重启kibana
docker restart d4c437270d84

# 访问kibana
http://172.16.39.2:5601/
```

##### Kibana控制面板

###### ![03](/Users/coco/Documents/Learn/ELK/img/03.png)

![04](/Users/coco/Documents/Learn/ELK/img/04.png)

##### 查看索引与分片

```shell
get _cat/indices?v  # 所有索引
get _cat/shards?v  # 所有分片
```

![05](/Users/coco/Documents/Learn/ELK/img/05.png)

##### 创建索引

```
命令语法：PUT 索引名{索引配置参数}
index 名称必须小写，且不能以下划线“_”，“-”，“+”开头

7.0版本之前默认5个主分片，之后默认1个主分片1个副分片
默认在创建索引时，会分配1个primary shard,1个replica shard。在es中，默认的限制是：如果磁盘不足15%不再分配replica shard；如果磁盘不足5%，不再分配任何primary shard。es中对shard的分布是有要求的，尽可能的保证paimary shard平均分布在多个节点上；replica shard会保证不和他备份的那个parimary shard在同一个节点上
```

###### 默认索引

```shell
put 索引名
eg:
	put index_test
```

![06](/Users/coco/Documents/Learn/ELK/img/06.png)

![07](/Users/coco/Documents/Learn/ELK/img/07.png)

![08](/Users/coco/Documents/Learn/ELK/img/08.png)

###### 指定创建

```shell
put 索引名
{
  "settings":{
    "number_of_shards":主分片数量,
    "number_of_replicas":副分片数量
  }
}
```

![09](/Users/coco/Documents/Learn/ELK/img/09.png)

##### 修改索引

```shell
# 索引一旦创建，primary shard不可修改，只能修改replica shard的数量
put 索引名/_settings
{
  "number_of_replicas":副分片数量
}
eg:
  put index_test2/_settings
  {
    "number_of_replicas":1   # 每个索引下，一个主分片，一个副分片
	}
```

![10](/Users/coco/Documents/Learn/ELK/img/10.png)

##### 删除索引

```shell
delete 索引名
eg:
	delete index_test2
```

