---
title: "kubernetes mysql"
date:  2024-03-13 16:33:32 +0800
categories: [kubernetes]
tags: [k8s]
---

##  配置

```ini
[mysqld]

default-time-zone = "+08:00"
max_connections = 500
```

```sql
SELECT @@global.time_zone;
show variables like "max_connections";
set global max_connections = 200;
```

```yaml
  mycnf: |
    [mysqld]
    max_connections=1000
    default-time-zone="+08:00"
    innodb_buffer_pool_size=51539607552
    innodb_buffer_pool_instances=8
    max_allowed_packet=524288000
    net_write_timeout=3600
    binlog_expire_logs_seconds=259200
    innodb_log_buffer_size=1073741824
    innodb_log_file_size=1073741824
    innodb_flush_method=O_DIRECT
    innodb_io_capacity_max=8000
    innodb_io_capacity=4000
    replica_parallel_workers=40
    log_bin=ON
    sql_mode=STRICT_TRANS_TABLES
    innodb_write_io_threads = 12
    innodb_read_io_threads = 12
```

##  集群

```sh
# 进入container
mysqlsh -uroot -p -h127.0.0.1

STOP GROUP_REPLICA;
START GROUP_REPLICA;

dba.dropMetadataSchema()
dba.configureInstance()
dba.checkInstanceConfiguration()

# 查看集群
var cluster = dba.getCluster()
cluster.status()
cluster.describe()
cluster.rescan()
cluster.dissolve()

cluster = dba.createCluster('devCluster')
# option {recoveryMethod: 'clone'} 当无法以增量的方式增加实例时（比如某些table没有主键）
cluster.addInstance('root@localhost:3330')
cluster.removeInstance('xxxx',{force:true})


mysqlrouter --bootstrap root@localhost:3310 -d mysqlrouter # 重新引导router

\help
# exit
\quit
```

## k8s 部署

### 单个实例

做持久化，新建 data，config pvc, 配置挂载在`/etc/mysql/conf.d/mysqld.cnf`

```yaml
- image: mysql:8.0.36
  env:
    - name: MYSQL_ROOT_PASSWORD
      value: xxxx
  volumeMounts:
    - name: "config"
      mountPath: "/etc/mysql/conf.d/mysqld.cnf"
      subPath: "mysql.conf.d"
    - name: "data"
  mountPath: "/var/lib/mysql"
  nodeSelector:
    workload: production
  volumes:
  - name: "config"
    configMap:
      name: mysql-config
  - name: "data"
    persistentVolumeClaim:
      claimName: nfs-mysql



```

```ini
[mysqld]
innodb_log_buffer_size=16777216
default-time-zone="+08:00"
log_bin=OFF
innodb_flush_method=O_DIRECT
```

### 集群

[helm install mysql operator](https://dev.mysql.com/doc/mysql-operator/en/mysql-operator-installation-helm.html)

```sh
helm install mycluster mysql-operator/mysql-innodbcluster \
    --set credentials.root.user='root' \
    --set credentials.root.password='sakila' \
    --set credentials.root.host='%' \
    --set serverInstances=3 \
    --set routerInstances=1 \
    --set tls.useSelfSigned=true
```

## FAQ

###  如何导出views

导出view create scripts

```sh
mysql -pxxxx -u root -h127.0.0.1  --skip-column-names --batch -e 'select CONCAT("DROP TABLE IF EXISTS ", TABLE_SCHEMA, ".", TABLE_NAME, "; CREATE OR REPLACE VIEW ", TABLE_SCHEMA, ".", TABLE_NAME, " AS ", VIEW_DEFINITION, "; ") table_name from information_schema.views' > /tmp/views.sql

```

###  mysql router 断开所有连接

> 如果用户向仅剩一个成员的现有集群添加新实例，MySQL Router 将断开与集群的所有连接。发生这种情况的原因是新实例存在于组复制元数据中，但尚未存在于集群的元数据中。MySQL Router 认为没有法定人数并断开连接。在集群的元数据中显示新实例后，可以重新连接

> mysql cluster 8.0.19 后元数据由1.x 升至 2.x ,需upgrade metadata 先，再部署高版本 router


## 失去法定人数

```sql
SELECT @@group_replication_local_address;
set global group_replication_force_members=@@group_replication_local_address;
SET GLOBAL group_replication_force_members="127.0.0.1:10000,127.0.0.1:10001";
```
> This procedure uses group_replication_force_members and should be considered a last resort remedy. It must be used with extreme care and only for overriding loss of quorum. If misused, it could create an artificial split-brain scenario or block the entire system altogether.

## 问题

1. Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND

集群中在`SECONDARY`只读节点强行SET global super_read_only = OFF ,修改表数据，如 truncate table，将导致group replication出现以上错误

如何修复

节点mysql将自动shutdown并无法重新加入集群，不断重启，应先移除集群 cluster.removeInstance ，节点mysql实例将不再discovery by cluster，此时可正常run，再重新加入，默认将以Incremental的 ReconverMethod 加入集群并恢复，可能部分table没有primary key，没有通过precheckconfig,所以 cluster.addInstance('xxxx:3306',{recoveryMethod: 'clone'}) 加入，先Drop table 再copy file.

2. 报Unkown variable ，配置问题

因pod没有销毁，pvc的内容将不变，即使改了k8s configmap 也无法同步本地config

> delete pod , mysql pod 中包含sidecar 和mysql， 即使 mysql 因错误重启，也无法同步configmap
> 找到pvc，找到config，vim 进入改

## 参考

[mysql router setting](https://dev.mysql.com/doc/mysql-shell/8.0/en/setting-up-innodb-cluster-and-mysql-router.html)

[mysql router 8.0.33 bug about only one remaining member](https://dev.mysql.com/doc/relnotes/mysql-router/8.2/en/news-8-2-0.html#mysql-router-8-2-0-bug)

[复制时统计](https://dev.mysql.com/doc/refman/8.0/en/replication-threads-monitor-worker.html)

[复制时线程](https://mysql.net.cn/doc/refman/8.0/en/source-thread-states.html)

[监控mysql集群](https://dev.mysql.com/doc/mysql-shell/8.0/en/monitoring-innodb-cluster.html#innodb-cluster-monitoring-recovery)

[组复制](https://dev.mysql.com/doc/refman/8.0/en/group-replication-primary-secondary-replication.html)

[使用 Performance_schema 实现 MySQL 组复制监控中的可观察性](https://www.mydbops.com/blog/observability-in-mysql-group-replication-monitoring-with-performance_schema/)

[view replication_status_full](https://gist.github.com/lefred/1bad64403923664a14e0f20f572d7526)

[MySQL 8 and Replication Observability](https://blogs.oracle.com/mysql/post/mysql-8-and-replication-observability)

[MGR最优化配置推荐](https://cloud.tencent.com/developer/article/1831605)

[MySQL 并行复制可以帮助我的从属服务器吗？](https://www.percona.com/blog/can-mysql-parallel-replication-help-my-slave/)

[Handling a Network Partition and Loss of Quorum ](https://dev.mysql.com/doc/refman/8.0/en/group-replication-network-partitioning.html)

[MySQL any way to import a huge (32 GB) sql dump faster?](https://dba.stackexchange.com/questions/83125/mysql-any-way-to-import-a-huge-32-gb-sql-dump-faster)
