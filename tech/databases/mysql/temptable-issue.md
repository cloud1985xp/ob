---
tags:
  - mysql
  - database
  - aurora
created: 2024-01-01
updated: 2025-01-23
status: active
---


New Temporary Table Behavior in Aurora MySQL version 3
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/ams3-temptable-behavior.html

Aurora MySQL 建議的連線與 Storage 上限
https://docs.aws.amazon.com/zh_tw/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Performance.html#AuroraMySQL.Managing.TempStorage

ISHIN 資料整理
https://docs.google.com/spreadsheets/d/12JdGjVaddfMk1o-m0I00AaXHi1OVDPEeUSFErmB1u4c/edit?gid=0#gid=0
Postmortem
https://wikib.aktsk.jp/pages/viewpage.action?pageId=384424795



一些當時用來測試的語法
```
select @@innodb_read_only,@@tmp_table_size,@@temptable_max_ram,@@temptable_max_mmap, @@internal_tmp_mem_storage_engine;

select @@innodb_read_only,@@aurora_version,@@aurora_tmptable_enable_per_table_limit,@@temptable_max_ram,@@temptable_max_mmap,@@tmp_table_size;

  

SET GLOBAL temptable_max_mmap = 1073741824 -- 1073741824
set GLOBAL temptable_max_ram = 16777216 -- 1073741824 / set to default
set GLOBAL temptable_max_ram = 1048576

SET temptable_max_mmap = 1048576
set temptable_max_ram = 1048576
set tmp_table_size= 1048576

  
SELECT `banners`.`id`, `banners`.`platform`, `banners`.`place`, `banners`.`priority`, `banners`.`announced_locales`, `banners`.`conditions`, `banners`.`start_at`, `banners`.`end_at`, `banners`.`created_at`, `banners`.`updated_at` FROM `banners` WHERE `banners`.`end_at` > '2024-07-22 10:40:00' AND `banners`.`start_at` <= '2024-07-22 10:40:00' AND (banners.announced_locales & 1 <> 0) AND (banners.platform & 1 <> 0) AND `banners`.`id` IN (SELECT `banner_availables`.`banner_id` FROM `banner_availables` WHERE (CONCAT('2024-07-22', ' ', `banner_availables`.`wday_start_at`) <= '2024-07-22 10:40:00' AND (`banner_availables`.`wday` = 0 OR (banner_availables.wday & 2 <> 0)) AND CONCAT('2024-07-22', ' ', `banner_availables`.`wday_start_at`) <= '2024-07-22 10:40:00' AND DATE_ADD(CONCAT('2024-07-22', ' ', `banner_availables`.`wday_start_at`), INTERVAL `wday_hours_to_end` HOUR) >= '2024-07-22 10:40:00' OR CONCAT('2024-07-22', ' ', `banner_availables`.`wday_start_at`) > '2024-07-22 11:40:00' AND (`banner_availables`.`wday` = 0 OR (banner_availables.wday & 1 <> 0)) AND CONCAT('2024-07-21', ' ', `banner_availables`.`wday_start_at`) <= '2024-07-22 10:40:00' AND DATE_ADD(CONCAT('2024-07-21', ' ', `banner_availables`.`wday_start_at`), INTERVAL `wday_hours_to_end` HOUR) >= '2024-07-22 10:40:00')) AND `banners`.`place` = 0

  
SHOW GLOBAL STATUS where Variable_name like '%tmp%'
SHOW VARIABLES LIKE "%temp%"
SHOW STATUS where Variable_name like '%tmp%'

show status like '%tmp%';
set cte_max_recursion_depth=4294967295;

WITH RECURSIVE cte (n) AS (SELECT 1 UNION ALL SELECT n + 1 FROM cte WHERE n < 5000000) SELECT max(n) FROM cte;

WITH RECURSIVE cte (n) AS (SELECT 1 UNION ALL SELECT n + 1 FROM cte WHERE n < 60000000) SELECT max(n) FROM cte;

WITH RECURSIVE cte (n) AS (SELECT 1 UNION ALL SELECT n + 1 FROM cte WHERE n < 240000000) SELECT max(n) FROM cte;

set tmp_table_size= 1048576
SET GLOBAL aurora_tmptable_enable_per_table_limit = 1

SHOW GLOBAL STATUS
SHOW PROCESSLIST
```


## References
AWS
https://docs.aws.amazon.com/zh_tw/AmazonRDS/latest/AuroraUserGuide/ams3-temptable-behavior.html
https://aws.amazon.com/cn/blogs/database/use-the-temptable-storage-engine-on-amazon-rds-for-mysql-and-amazon-aurora-mysql/
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/ams3-temptable-behavior.html
MySQL

https://dev.mysql.com/doc/refman/8.0/en/internal-temporary-tables.html
https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_tmp_table_size
https://dev.mysql.com/doc/refman/8.4/en/innodb-temporary-tablespace.html