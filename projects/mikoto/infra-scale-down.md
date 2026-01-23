---
tags:
  - mikoto
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---


## Merge Player DB

- Upgrade from r5.large -> r6g.xlarge (30min)
- Dump 5min per shards
	- 5 * 2+10mins =~ 30mins
	- used 44GB diskspace for 1 shard
- Import (using xlarge), 
	- importer is c6i.2xlarge, CPU: 8%, maybe could be downgraded
	- 2024-09-06 07:28:18 ~  2024-09-06 09:15:27
	- 110min * 2 
- Import (using 2xlarge)


Dump
```
source envfile
CONFIG_FILE=prod.conf
DUMP_LOG_FILE=dump.log

mydumper --defaults-file $CONFIG_FILE -o $DUMP_DIR -t 16 --logfile $DUMP_LOG_FILE -v 3
```

Import
```
RESTORE_LOG_FILE=restore.log
myloader --defaults-file $CONFIG_FILE -d $DUMP_DIR -v 3 -B $PLAYER_DB_NAME -t 16 --logfile $RESTORE_LOG_FILE


source envfile
CONFIG_FILE=prod.conf
PLAYER_DB_NAME=mikoto_repo_player11_test
myloader --defaults-file $CONFIG_FILE -d $DUMP_DIR -v 3 -B $PLAYER_DB_NAME -t 16 --logfile $RESTORE_LOG_FILE

myloader --defaults-file $CONFIG_FILE -d $DUMP_DIR -v 3 -B $PLAYER_DB_NAME -t 16 --logfile $RESTORE_LOG_FILE
```

2nd test
- r5.2xlarge
- started: 2024-09-08 07:31:51 UTC
- ended: 2024-09-08 08:40:58 UTC

Modify instance type
r5.2xlarge to r5.large: 04:42 ~ 16:50 (8mins)
r5.large to r6g.2xlarge: 17:10 ~ 17:18 (8mins)




RDS Downgrade to r6g.large (15mins)

EC2
- AS: downgrade from c5.2xlarge to c5.large

Elasticache
- Change to g type
- All region: combine mem


migrator1
- i-04908f6299ddfa90c
- repo05, repo09 -> repo01

migrator2
i-08fae4a797fdc1e7c

migrator3
i-043f42525b6435045

migrator4
i-082810ff1318b9b86)

## US

EC2
Delete staging: (auth/app/avalon)
- prod-app * 2
- prod-auth * 2
- info-ecs * 2

RDS: 
Delete common (global) database, merge into player01


- player01
- player02


(Upgrade to r6g.large?)

prod-app 4 -> 2

## BQ Sender
TBD