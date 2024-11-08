---
title: "seatunnel"
date:  2024-08-06 09:54:30 +0800
categories: [cdc]
tags: [cdc,myql]
---

## seatunnel

[seatunnel](https://github.com/apache/seatunnel)

[seatunnel-doc](https://seatunnel.apache.org/docs/2.3.6/about)

### source
exactly-once
> 如果数据源中的每条数据仅由源向下游发送一次，我们认为该source connector支持精确一次（exactly-once）。

### sink

exactly-once
> 当任意一条数据流入分布式系统时，如果系统在整个处理过程中仅准确处理任意一条数据一次，且处理结果正确，则认为系统满足精确一次一致性。

## seatunnel-web

[seatunnel-web](https://github.com/apache/seatunnel-web)

```sh
cd seatunnel-web
build.sh code
```

## Bugs
1. seatunnel-ui

`$npm run build:prod` 提示 `$route` 不存在,需修改 `\seatunnel-ui\node_modules\vue-router\dist\vue-router.d.ts`

vue3 style
```
- declare module '@vue'

+ declare module '@vue/runtime-core'
```

2. download_datasource.sh not found

git clone into window OS, file end of LRLF,  can not be execute in linux.

```sh
sed -i 's/\r//' apache-seatunnel-web-1.0.0-SNAPSHOT/bin/seatunnel-backend-daemon.sh
sed -i 's/\r//' apache-seatunnel-web-1.0.0-SNAPSHOT/bin/download_datasource.sh
```

3. 2.3.3 java.util.LinkedHashMap cannot be cast to java.util.ArrayList

~~had been fixed in 2.3.5~~ 2.3.5 依然需要将重新编译修改后的`seatunnel-config-base.jar`放入libs文件夹

[workaround](https://github.com/apache/seatunnel/issues/5700)


`seatunnel-config-base`

```xml
<!-- just add this line to remove the inner class -->
<exclude>com/typesafe/config/impl/ConfigParser$ParseContext.class</exclude>
```


```config
env {
  parallelism = 1
  job.mode = "STREAMING"
  checkpoint.interval = 10000
}

source {
  MySQL-CDC {
    base-url = "jdbc:mysql://xxxx:3306/e3plus_stock_integrate"
    username = "root"
    password = "xxx"

    table-names = ["e3plus_stock_integrate.stk_stock_logic_account"]
  }
}

sink {
    jdbc {
        driver = "com.mysql.cj.jdbc.Driver"
        url = "jdbc:mysql://xxxx:3306/e3plus_stock_integrate?useUnicode=true&characterEncoding=UTF-8&rewriteBatchedStatements=true"
        user = "root"
        password = "xxx"

        # generate_sink_sql = true
        # You need to configure both database and table
        database = e3plus_stock_integrate
        table = stk_stock_logic_account
        primary_keys = ["id"]
        # field_ide = UPPERCASE
        schema_save_mode = "CREATE_SCHEMA_WHEN_NOT_EXIST"
        data_save_mode="APPEND_DATA"
    }
}


```


```sh
kubectl  -n mysql exec xx -c mysql   -- mysqldump  --disable-keys  --set-gtid-purged=OFF   --single-transaction  --no-data  -h127.0.1.0 -uroot  -p xx  > xx.sql
```


```sql
-- 排除不兼容的tables, 列名带# ，表名带 .
SELECT * FROM information_schema.columns where table_schema = 'xxxx' and (column_name like '%#%' OR  table_name like '%.%');

```

1. snapshot splitreader 阶段暂停task，savepoint 保持offset，缓存中的queue是否存在？
内存
2. seatunnel 退出的原因
cluster miss `-d`, Ctrl+c 退出
3. 恢复任务
即使data_save_mode=drop_data，有checkpoint的情况下依然可以增量




[Incremental Snapshots in Debezium](https://debezium.io/blog/2021/10/07/incremental-snapshots/#:~:text=When%20a%20table%20snapshot%20is%20requested%2C%20then%20Debezium,size%20as%20prescribed%20by%20the%20incremental.snapshot.chunk.size%20configuration%20option)


```ini
     debezium{
       include.schema.changes = false
     }

```
