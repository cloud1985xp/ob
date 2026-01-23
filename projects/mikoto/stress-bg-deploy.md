---
tags:
  - mikoto
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---

# TODO/Checklist

- 確認 production rds cluster quota for BG deployment
- Confirm Blue cluster has binlog_format updated
- How to perform validation query in all clusters
- Make sure the db migration Jenkins job should be updated to v3.6.0 and run the common DB migration

# mikoto 會執行的步驟：
會進行包括但不限於以下操作：
- v3.5.0 stress 測試 with production player data -> 已完成
- 跑 make_bg_deploy 開 blue green deployment <- 這個比較花時間，我應該會在中午前就丟下去執行

13:30 後
- [x] 確認 bg deployment、parameter group or reboot green
- [x] 執行 charset migration script (mikoto 是用 codebuild)、check healthy metrics、紀錄花費時間 -> 36 minutes 20 seconds
  - check replica lag metrics (Ref: https://docs.aws.amazon.com/zh_tw/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMonitoring.Metrics.html)
  - Run `SHOW REPLICA STATUS`
- [x] 執行 charset validation
- [x] 執行 warmup db (應該還是會 dump all tables)、紀錄花費時間


- v3.6.0 stress 測試(待確認，v3.6.0 的 masterdata 是否能拿來跑測試)
  - [x] bg_switch 切換到 aurora3
  - [x] 部署 v3.6.0 版本，紀錄 player db migration 花費時間 -> 6 minutes
- 執行 stress test 比較 v3.5 & v3.6 (aurora2 & aurora3) 的負荷變化
- [] v.3.6.0 locust 腳本測試k

跑 make_bg_deploy 開 blue green deployment (35mins)
確認 parameter group or reboot green
跑 charset migration script (mikoto 是用 codebuild)、check
跑 warmup db (應該還是會 dump all tables)
部署新版本 (測試 migration 時間)

# Timeline
make_bg_deploy 11:45 started
make_bg_deploy 12:25 completed
40mins

reboot instane: 13:45 started
                13:47 completed
2mins

charset migration
36 minutes 20 seconds


warmupdb 14:57 started
         15:11 completed

14 minutes

================ Before Maintenance ======================

switch   15:15 started
         15:17 completed
2 minutes

v3.6.0 schema migration:
6 minutes

# Scripts to Dump Tables

```
dbname="mikoto_player_repo1"
dbhost="stress-rds-player01dbcluster-7avfjnnt5c9m-green-qo3ckj.cluster-cebmb5lwi61r.us-west-2.rds.amazonaws.com"

db_user=$(aws ssm get-parameter --name "mikoto-gl-prod.rds.username" --output json| jq -r '.[].Value')
db_password=$(aws ssm get-parameter --name "mikoto-gl-prod.rds.password" --with-decryption --output json| jq -r '.[].Value')

echo "Warmup DB: ${dbname} by dumping all tables..."

date
mysql -u $db_user -p$db_password --host $dbhost $dbname -sNe 'show tables' | \
while read table
do
  echo ${table}
  mysqldump -u $db_user -p$db_password --host ${dbhost} --skip-lock-tables ${dbname} ${table} > /dev/null &
done; wait
date
```

