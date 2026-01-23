---
tags:
  - business
  - planning
created: 2024-01-01
updated: 2025-01-23
status: active
---


stress 測試的數據我紀錄在這個 sheet
https://docs.google.com/spreadsheets/d/16xgEegCM53wkx8qkCXjNBHj47eFCtjSu-Y5aj8nBngs/edit?gid=501732310#gid=501732310

幾個這次測試的目標

Redis 更換成 Valkey
更換前後 cluster 的 CPU Loading 並無顯著變化 (4.9% -> 4.7%)

Memcached 降級
stress 測試無法驗証
stress 是用 1/32 的規模，使用 t4g type 
production ri 更換僅是 memory storage 的變動，直接從 cloudwatch metric 觀察 freeable memory 是否足夠

scaling_rds_read_replica 腳本能否正常運作
測試結果可以正常運行，但啟動 scaling schedule 需等後一段時間 (> 5mins)
master db cluster 才會開始 scale-out
新建立的 db instance 需花約 > 30mins 才建立完成

modify_user_db_before_maintenance 腳本能否正常運作
測試結果可以正常運行
執行時間約花費 25min

modify_user_db_under_maintenance 腳本能否正尚運作
測試結果可以正常運行
執行時間約花費 15min

RDS Aurora cluster enable io optimize
無法透過 cli 執行，是用 web console 去更新
執行時間約花費 5m


