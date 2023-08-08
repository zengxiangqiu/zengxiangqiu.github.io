---
title: "ClickHouse"
date:  2023-06-28 11:18:13 +0800
categories: [database]
tags: [clickhouse]
---

https://github.com/ClickHouse/ClickHouse/blob/master/LICENSE


##  Connect ClickHouse

下载[ClickHouse 驱动](https://github.com/ClickHouse/clickhouse-odbc)

下载[MDAC](https://www.microsoft.com/en-us/download/details.aspx?id=21995)

打开ODBC管理数据源（64位），选择ClickHouse驱动（UniCode）添加demo，`url=https://play.clickhouse.com:443/`,`username=explorer或者play`，参考[官网](https://clickhouse.com/docs/zh/getting-started/playground)

打开 ODBCTest（Unicode,amd64），conn->full connect->选择之前添加的数据源，`select '1'`,Ctrl+E, SuccessCode=0, Ctrl+R，Get all data;


##  阿里云 ClickHouse

[通过HTTPS协议连接ClickHouse](https://help.aliyun.com/document_detail/393349.html?spm=a2c4g.300433.0.0.38ed2fcbnaJyYY)
