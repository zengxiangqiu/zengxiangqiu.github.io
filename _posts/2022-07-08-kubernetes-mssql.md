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
