---
tags:
  - ishin
  - sre
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# ISHIN Aurora3

Log DB Cluster? ⇒ yes

Migrate to Aurora3 (MySQL 8), required DB engine changes, need blue/green deployment

Migrate to use utf8mb4 (original utf8 is equivalent to utf8mb3 in mysql 8)

Need to modify parameters to allow MIXED binlog form

Since MySQL 8 removed query cache, cause performance issue, so that master DB needs scale-out/up

Log DB change charset has issue, 

Because many tables are too large to duplicate temp table, it cannot dynamic convert, so keep old tables and create new table with utf8mb4

DDL optmize

先執行 aurora 2→3 升級，看哪些 table 需要做 optimize

v5.15

建立 parameter group

建立 blue/green deployment

檢查 schema

The reason we do blue/green is because we want to convert charset to utf8mb4

And this part is optional for log DB

[https://miro.com/app/board/uXjVM1clbvM=/?share_link_id=520456010503](https://miro.com/app/board/uXjVM1clbvM=/?share_link_id=520456010503)