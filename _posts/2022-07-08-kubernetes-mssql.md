---
title: "kubernetes mssql"
date:  2022-07-08 12:22:54 +0800
categories: [k8s]
tags: [mssql]
---

## Troubleshot

1. provider: SSL Provider, error: 31 - Encryption(ssl/tls) handshake failed

修改 dockerfile

```dockerfile
FROM base AS final
RUN sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1/g' /etc/ssl/openssl.cnf
RUN sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1/g' /usr/lib/ssl/openssl.cnf
WORKDIR /app
COPY --from=publish /app/publish .
```


[SQL Server 2012 SP1（11.0.3000.00 - 2012 年 11 月)](https://learn.microsoft.com/zh-cn/troubleshoot/sql/releases/download-and-install-latest-updates)

[Extended Security Updates Ends in 1 year and 11 months (08 Jul 2025)](https://endoflife.date/mssqlserver)

[Clarify supported Microsoft SQL Server versions](https://github.com/sequelize/sequelize/issues/13838)

> Sequelize does not need to claim support for versions 2012, 2014 if we can't test that in the pipeline.

[filter sp_who2](https://www.stevefenton.co.uk/blog/2018/07/sql-server-filter-and-sort-records-from-sp_who2/)
