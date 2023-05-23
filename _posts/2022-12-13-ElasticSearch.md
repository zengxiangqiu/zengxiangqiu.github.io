---
title: " ElasticSearch"
date: 2022-12-13 14:54:29 +0800
categories: [elastic]
tags: [elastic]
---

## 概念

> 我们使用的术语 对象 和 文档 是可以互相替换的
> shard = hash(routing) % number_of_primary_shards
> routing 是一个可变值，默认是文档的 \_id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置

在 hits 数组中每个结果包含文档的 \_index 、 \_type 、 \_id ，加上 \_source 字段。这意味着我们可以直接从返回的搜索结果中使用整个文档。

\_shards 部分告诉我们在查询中参与分片的总数

一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

同时你可以使用 from 和 size 参数来分页：

索引是由段（Segment）组成的，段存储在硬盘（Disk）文件中，段不是实时更新的，这意味着，段在写入磁盘后，就不再被更新

`curl -X GET 'http://localhost:9200/lsd-_0/_search?q=filebeat_fields_source:nginx&pretty'`

Lucene 的段是分别存储到单个文件中的。因为段是不可变的，这些文件也都不会变化，这是对缓存友好的，同时操作系统也会把这些段文件缓存起来，以便更快的访问。

```bash
# 硬盘
df -h

lsblk
```

一个 Elasticsearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性

Elasticsearch 默认按照相关性得分排序

索引 —— 保存相关数据的地方。 索引实际上是指向一个或者多个物理 分片 的 逻辑命名空间 。

倒排索引包含一个有序列表，列表包含所有文档出现过的不重复个体，或称为 词项 ，对于每一个词项，包含了它所有曾出现过文档的列表

一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，然后像 执行分布式检索 提到的那样，合并每个分片的结果到一个全局的结果集。

文档更新和删除也会导致大量的合并数，因为它们会产生最终需要被合并的段 碎片

映射, 就像数据库中的 schema ，描述了文档可能具有的字段或 属性 、每个字段的数据类型—比如 string, integer 或 date —以及 Lucene 是如何索引和存储这些字段的。

```bash
docker run -d -p 9201:9200 -p 9301:9300 --name=es02 -e bootstrap.memory_lock=true -e ES_JAVA_OPTS="-Xms2g -Xmx2g" -v /var/lib/es02:/var/lib/elasticsearch -v /var/log/es02:/var/log/elasticsearch -v /root/es02.yml:/usr/share/elasticsearch/config/elasticsearch.yml --cap-add=IPC_LOCK --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 docker.elastic.co/elasticsearch/elasticsearch:6.7.2


docker run -d -p 9200:9200 -p 9300:9300 -e bootstrap.memory_lock=true -e ES_JAVA_OPTS="-Xms2g -Xmx2g" -v /var/lib/elasticsearch:/var/lib/elasticsearch -v /var/log/elasticsearch:/var/log/elasticsearch -v /root/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml --cap-add=IPC_LOCK --ulimit nofile=262144:262144 --ulimit memlock=-1:-1 docker.elastic.co/elasticsearch/elasticsearch:6.7.2
```

> transport.bind_host, which the node should bind in order to listen for incoming transport connections
transport.publish_host, which the node can be contacted by other nodes

写入 index， es 写入 translog 和 in-memory, \_refresh 将缓存区写入 lucene Segment(内存), \_flush 将 Segment 写入 disk。

POST `/_cluster/reroute?retry_failed=true` 重启重试，5 次失败后会停止

POST `/_flush/synced` 自动分配

POST `/_cluster/allocation/explain?pretty` 分配解析

POST `/_cluster/settings` 开启自动分配

```json
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

## kopf

```docker
docker run --name=kopf \
        --hostname=kopf \
        -p 80:80 \
        --env=KOPF_SERVER_NAME=grafana.dev \
        --env=KOPF_ES_SERVERS=192.168.20.95:9200 \
        --restart=always \
        lmenezes/elasticsearch-kopf
```

## cerebro

适用于 es 2.0+ 之后

```bash
# create brige network
docker network create es

# run cerebro
docker run -d  --network=es --name=cerebro  -e http.port=80  -p 80:80 lmenezes/cerebro
```


[runlike](https://github.com/lavie/runlike)

[Kubernetes-elasticsearch-nfs 集群部署](https://juejin.cn/post/7210338718312955961)

[How to Delete Elasticsearch Unassigned Shards in 4 Easy Steps](https://www.cyberithub.com/how-to-delete-elasticsearch-unassigned-shards)

[autodiscover](https://www.elastic.co/guide/en/beats/filebeat/6.8/configuration-autodiscover.html)
