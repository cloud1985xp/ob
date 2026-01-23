---
tags:
  - mysql
  - database
  - aurora
created: 2024-01-01
updated: 2025-01-23
status: active
---

目前的 survey 結合上次 aws support 說的 purge function  

- mysql 執行刪除不會馬上真的刪掉 physical 的 data 和索引紀錄
- 而是存成像 undo log (保有 rollback 的可能)
- purge function 就是定期地去清除 undo log
- 在 parameter group 中我們可以去設定 purge function 的相關參數，例如配置的 threads 數、purge max lag 值、purge batch size (undo log頁數)…等
- 上面 mysql error log 訊息，出現 undo log 中有 invalid 的資料 (ERROR: Found invalid undo record! Undo record len=18446744073709551397, rec_offset=219, next_rec_offset=0 The orginal rec trx id:161231930183)，有可能是 aws support 說的有關 user_mission 那個 index 的資料造成
- error 裡中有提到可以透過調整 innodb_force_recovery 來修復
- 從 innodb_force_recovery 的[設定說明](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html), 其中

- 設成 (2) 以上是可以阻止 purge function 的，
- 或是設成 (3) 在 recovery 時不處理 rollback
- 又或是 (5) 完全不看 undo log

- 但這個參數在 aurora 上 `我們不能自己調整` => 可能就是那時候 aws support 說要由他們變更的設定
- 系統 zerodown reboot -> 啟動後要做 recovery，但會去從 undo log 去對 transaction 做 rollback，但 undo log invalid -> 繼續 zerodown time reboot
- 然後它一直 reboot 大概是又一直向系統調用 file descriptor，所以 error log 到後面我有看到還出現 file descriptor 的 error
- 所以 weekly user sync 其實一開始還勉強能 sync 資料，雖然它一直重啟但還是能 query 資料，只是會很慢，但到後面沒有 file descriptor 了，process 就直接整個卡住
- 後來試著找有沒有直接清掉 undo log 的方法，找到上面那個 query，但這時候我已經把有問題的 db 刪除了，無法測試

Ref:  
[https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)  
[https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html)  
[https://www.cnblogs.com/Miac/p/13269980.html](https://www.cnblogs.com/Miac/p/13269980.html)  
[https://www.cnblogs.com/glon/p/6728380.html](https://www.cnblogs.com/glon/p/6728380.html)  
[https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)

### Tablespace
https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html
https://stackoverflow.com/questions/70313924/mysql-how-to-delete-the-undo-log-with-the-cli-on-ubuntu
https://stackoverflow.com/questions/42628687/undo-log-error-no-more-space-left-over-in-system-tablespace-for-allocating-undo/51667151#51667151

https://dev.mysql.com/doc/refman/8.0/en/set-variable.html
https://dev.mysql.com/doc/refman/5.7/en/innodb-purge-configuration.html
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.ParameterGroups.html
https://stackoverflow.com/questions/62740079/mysql-undo-log-keep-growing
https://bugs.mysql.com/bug.php?id=110598
https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_force_recovery


