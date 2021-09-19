### ELK

#### 一、ELK介绍

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

#### 二、ELK安装(Centos 7)

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
docker run -it -d --name kibana --link es:es -p 5601:5601 --restart=always kibana:7.6.2

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

###### Kibana控制面板

###### ![03](/Users/coco/Documents/Learn/ELK/img/03.png)

![04](/Users/coco/Documents/Learn/ELK/img/04.png)

#### 三、索引

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

#### 四、文档

##### 新增Document

```
Elasticsearch有自动识别机制。如果增加的document 对应的index不存在，自动创建index;如果index存在，则使用现有的index
```

###### 1、PUT方式

```shell
# 此操作为 手工指定id 的新增方式
put 索引名/_doc/唯一id{字段名:字段值}   # 不存在则新增，存在则“全量”替换
```

![11](/Users/coco/Documents/Learn/ELK/img/11.png)

![12](/Users/coco/Documents/Learn/ELK/img/12.png)

```shell
put 索引名/_create/id          # 强制新增，如果不存在就新增，存在则报错
```

![13](/Users/coco/Documents/Learn/ELK/img/13.png)

###### 2、POST方式

```
POST方式，自动生成id
语法：
	POST 索引名/ID    # 此处虽然指定了id值，但是id依然自动生成
	POST 索引名/_doce  # 推荐
```

![14](/Users/coco/Documents/Learn/ELK/img/14.png)

##### 查询Document

```shell
# 查询全部
get 索引名/_search
eg：
	get index_test/_search
	
# 查询单数据
get 索引名/_doc/id
```

![](/Users/coco/Documents/Learn/ELK/img/15.png)

```shell
# 批量查询，推荐
get 索引名/_mget
{
  "docs":[
    {
      "_id":id_01
    },{
      "_id":id_02
    }
  ]
}
```

![16](/Users/coco/Documents/Learn/ELK/img/16.png)

##### 修改Document

```shell
1、使用PUT可以实现全量替换，进行修改。同上
2、post方式更新
	只更新Docment中部分字段，这种更新方式也是标记原有数据为delete状态，创建一个新的Document数据，将新的字段和未更新的原有字段组成这个新的Document，并创建。对比put的全量替换，只是操作方便，底层执行几乎没有区别
```

![17](/Users/coco/Documents/Learn/ELK/img/17.png)

##### 删除Document

```shell
	Elasticsearch中执行删除操作时，es先标记Documnet为delete状态，而不是直接物理删除，当es存储空间不足或工作空闲时，才会执行物理删除操作。标记为delete状态的数据不会被查询搜索到
	
语法：
	delete 索引名/_doc/id
```

![18](/Users/coco/Documents/Learn/ELK/img/18.png)

##### bulk 批量增删改

```shell
四种行为：
create：强制创建，相当于PUT索引名/_create/id。主键必须有
index：普通创建，相当于创建Document或全量替换
update：更新操作，相当于POST 索引名/_update/id
delete：删除操作
```

![19](/Users/coco/Documents/Learn/ELK/img/19.png)

![20](/Users/coco/Documents/Learn/ELK/img/20.png)

#### 五、分词器

##### 分词器介绍

###### Standard analyzer

```shell
Standard analyzer 是es中默认的分词器。是处理英语语法的分词器，切分过程中不会忽略停止词(the,a,an...),会进行单词的大小转换、过滤连接符(-)或括号等常见符号

语法：
  get _analyze
  {
    "text":"要被拆分的文本",
    "analyzer":"哪个分词器"
  }
```

![21](/Users/coco/Documents/Learn/ELK/img/21.png)

###### Simple analyzer

```
简单分词器，就是将数据切分为一个一个的单词，不会考虑语法。使用少
```

###### Whitespace analyzer

```
空白分词器，遇到空格就拆
```

###### Language analyzer

```
语言分词器，英语语法进行拆分，会忽略停止词、转换大小写、单词复数转换、时态转换等，与Standard analyzer类似
```

##### 安装中文分词器(Docker)

```shell
1、进入容器
docker exec -it es /bin/bash

2、在线安装IK
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.11.2/elasticsearch-analysis-ik-7.11.2.zip

# 在线安装可能会遇到Exception in thread "main" java.net.UnknownHostException: github.com
是因为虚拟机关闭了防火墙，需要重启docker

2、离线安装IK
# 本机mac下载elasticsearch-analysis-ik-7.11.2.zip,然后上传到虚拟机
scp Downloads/elasticsearch-analysis-ik-7.11.2.zip root@172.16.39.2:/opt
# 再从虚拟机上传到docker容器
docker cp /opt/elasticsearch-analysis-ik-7.11.2.zip 容器id:/usr/share/elasticsearch/plugins
# 解压
mkdir ik
unzip -d ik elasticsearch-analysis-ik-7.11.2.zip

3、重启容器
docker restart es
```

##### 配置词条(我没成功)

```
1、查看容器中配置词条的路径
docker exec -it es /bin/bash
cd /usr/share/elasticsearch/config/analysis-ik  # analysis-ik就是目录，里面的文件是乱码的

2、把analysis-ik的东西cp到虚拟机里任何目录
docker cp 05d931713007:/usr/share/elasticsearch/config/analysis-ik ./

3、进入虚拟机里的analysis-ik，配置词条
vim main.dic
在里面编写要添加的词条

4、再把虚拟机里的analysis-ik/main.dic放回到es容器中
docker cp ./main.dic 05d931713007:/usr/share/elasticsearch/config/analysis-ik

5、重启容器
docker restart es
```



##### IK介绍

```
ik提供两种分词器，分别是ik_max_word和ik_smart
ik_max_word：会将文本做最细粒度的拆分
ik_smart：会做最粗粒度的拆分
```

![22](/Users/coco/Documents/Learn/ELK/img/22.png)

#### 六、Mapping

```
mapping在es中是非常重要的额概念，决定了一个index中的field使用什么数据类型存储，使用什么分词器进行解析，是否有子字段等。
默认情况下，所有的text类型属性都是使用standard分词器
强调：
	1、类型必须记住
	2、text和keyword区别
	3、mapping一经生效不允许修改，所以创建索引时直接指定mapping关系
```

##### Mapping核心数据类型

```shell
# 只有text类型才能被分词
文本(字符串)：text				形式："数据类型"
整数：byte short integer long      形式：1000
浮点数：float double   形式：12.2    
布尔：boolean      形式：true/false
日期：date					形式：2021-09-19
数组：array      形式：{a:["value1","value2"]}
对象：object     形式：{a:{"key1":"value1"}} 
不分词的字符串(关键字)：keyword
```

![25](/Users/coco/Documents/Learn/ELK/img/25.png)

```shell
# 注意：如果第一次指定的“shuzu”是数组类型，那么在此使用“shuzu”这个field时，es会默认认为“shuzu”是数组的类型，即使不在同一个document；如果不在同一个index，没关系
```

![26](/Users/coco/Documents/Learn/ELK/img/26.png)

##### 查看mapping

```shell
get 索引名/_mapping
eg:
	get index_01/_mapping
```

![27](/Users/coco/Documents/Learn/ELK/img/27.png)

##### 自定义mapping

###### 创建索引自定义mapping

```shell
# 在创建index时才能自定义mapping
put 索引名
{
  "mappings":{
    "properties":{
      "字段名":{
        "type":"数据类型",
        "analyzer":"分词器"
      }
    }
  }
}
```

![28](/Users/coco/Documents/Learn/ELK/img/28.png)

###### 修改mapping

```shell
put 索引名/_mapping
{
  "properties":{
    "字段名":{
      "type":"数据类型",
      "analyzer":"分词器"
    }
  }
}
```

![29](/Users/coco/Documents/Learn/ELK/img/29.png)

###### 测试mapping分词

```shell
get 索引名/_analyze
{
  "field":"字段名",
  "text":"文本"
}
```

![30](/Users/coco/Documents/Learn/ELK/img/30.png)

#### 七、Search

```shell
# 测试数据:
post index_02/_bulk
{"create":{"_index":"index_02","_id":"101"}}
{"name":"name_01","age":11,"address":"上海浦东","desc":"我要钱啊"}
{"create":{"_index":"index_02","_id":"102"}}
{"name":"name_02","age":12,"address":"上海浦东","desc":"我要钱啊"}
{"create":{"_index":"index_02","_id":"103"}}
{"name":"name_02","age":13,"address":"上海浦东","desc":"我要钱啊"}
{"create":{"_index":"index_02","_id":"104"}}
{"name":"name_03","age":14,"address":"上海浦东","desc":"我要钱啊"}
{"create":{"_index":"index_02","_id":"105"}}
{"name":"name_03","age":12,"address":"上海浦东","desc":"我要钱啊"}
```

##### query

###### 查询所有数据match_all

```shell
get 索引名/_search
{
  "query":{
    "match_all": {}
  }
}
```

![31](/Users/coco/Documents/Learn/ELK/img/31.png)

###### match

```
get 索引名/_search
{
  "query":{
    "match": {
      "字段名":"内容"
    }
  }
}
```

![32](/Users/coco/Documents/Learn/ELK/img/32.png)

###### match_phrase

```shell
# 短语检索。要求查询条件必须和具体数据完全匹配才能搜索到。其特征：
	1、对搜索条件进行拆词
	2、把拆词当作一个整体，整体去索引(存储内容被拆词后的结果)中搜索，必须严格匹配才能查询到
get 索引名/_search
{
  "query":{
    "match_phrase": {
      "desc":"龙熊"
    }
  }
}
# 能匹配到“虎龙熊”，但是匹配不到“龙虎熊”，因为在我们的搜索条件中，龙在熊前并紧挨
```

![33](/Users/coco/Documents/Learn/ELK/img/33.png)

###### match_phrase_prefix

```
能匹配到 被拆分后的词条 的前缀就算匹配上了
```

![34](/Users/coco/Documents/Learn/ELK/img/34.png)

![35](/Users/coco/Documents/Learn/ELK/img/35.png)

###### range

```
大于gt，小于lt，大于等于gte，小于等于lte
```

![36](/Users/coco/Documents/Learn/ELK/img/36.png)

###### 多条件复合搜索

```
must：&& 逻辑与
must：！ 逻辑非
should：|| 逻辑或
```

![37](/Users/coco/Documents/Learn/ELK/img/37.png)

##### 排序

```shell
get 索引名/_search
{
  "query":{
    ....
  },
  "sort":{
    "根据哪个字段排序":{
      "order":"是逆序还是顺序"
    }
  }
}
```

![38](/Users/coco/Documents/Learn/ELK/img/38.png)

##### 分页

```shell
get index_02/_search
{
  "query":{
    ......
  },
  "sort":{
  	......
  },
  "from":起始下标,
  "size":每页大小
}
```

![39](/Users/coco/Documents/Learn/ELK/img/39.png)

##### 高亮

```shell
# 并不代表所有查询到的结果都有数据
get index_02/_search
{
  "query":{
    "match": {
      "name":"name_01"
    }
  },
  "highlight":{
    "fields":{
      "字段名":{
        "fragment_size":1,
        "number_of_fragments":1
      }
    },
    "pre_tags":"前缀",
    "post_tags":"后缀"
  }
}
```

![40](/Users/coco/Documents/Learn/ELK/img/40.png)

#### Logstash

##### docker安装

```python
# 下载
docker pull logstash:7.6.2

# 启动
docker run -it -p 4560:4560 --name logstash --restart=always -d logstash:7.6.2

# 修改配置
docker exec -it logstash /bin/bash
vi /usr/share/logstash/config/logstash.yml
hosts: [ "http://172.16.39.2:9200" ] # 修改为es的ip

# 修改输入输出配置
vi /usr/share/logstash/pipeline/logstash.conf
"""
input {
        tcp {
                mode => "server"
                port => 4560
        }
}
filter {
}
output {
        elasticsearch {
                action => "index"
                hosts => "172.16.39.2:9200"
                index => "test_log"
        }
}
"""
```

