---
tags:
  - mikoto
  - sre
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# MIKOTO

# Mikoto 1005

有關 infra 工作的近況交接

- 目前 infra 已經更跟日版最新，
- 會去檢查日版的 tag，看看是否有更新，然後開始準備作業
- Act: 確認 tag or 確認怎麼追蹤 tag

接下來可做的 Improvement

- service account keys 的管理、Rotation
- Update datadog agent

可以做的練習

- 直接建立一組 new environment
- 在新環境上實作 loading test
- update fluend to use the plugin of biqquery sender
- build k8s to transfer logs to bigquery

部署會議得到的資訊

- Info (Announcement)

- 不同於 ishin，公告瀏覽是沒有 no game log 的，只有 alb log

- common Info 是 prod/staging 共用的., 所以 announce 是在 dev 環境共用，因為只是一些 file

- Auth ALB -> Internal ALB

上面運行的 application，是使用 Dynamo DB (存放 registration info, friendship records 等)

- Game App ALB

app task(nginx, sever-app, datadog, fluentd)

- Admin - Avalon
- Admin - Avalon Phoenix
- Batch Cluster

上面運行壓制戰的 Sync Service -> 有獨自的 VPC (in US) sync data to redis of each Region

- Jenkins

遇到 info image 太久沒 build，在 sync efs 花太多時間，

SKIP_NETWORK=yes

UserShard:

as:12

us:8

eu:4

這次加了 Guild DB (01,02)

但這是未來才開的功能，所以先用 t2 type

但太小了，無法啟用 performance insight 功能

所以修改 cfn 的判斷，先跳過 guild DB (不啟用 Perf.insight)

Spot Instance 數量

GCP

pubsub

- push

- pull(o) <- DataFlow

CloudWatch / datadog

- 跟 ISHIN 不同，每個環境都是獨立的 vpc，common stack 是一些共用的 util
- 跑 migration 也是要重新 build image 的，因為
- 一些 ENV 是在 build time 就決定的，例如 SHARD_NUM，是直接設定在 codebuild 裡
- 在 codebuild - fluentd project 裡有 BIGQUERY_SENDER_HOST

[KPT 2021Q1Q2](MIKOTO/KPT%202021Q1Q2%20e1e197ac73154c15acc425d4e4e0b079.md)