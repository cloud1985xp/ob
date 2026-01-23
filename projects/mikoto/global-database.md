---
tags:
  - mikoto
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---


https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-disaster-recovery.html

## Consider Isolation Level

Isolation Level Limit when Writing Forward Enabled
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-write-forwarding-ams.html

https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.IsolationLevels.html

## Write Forwarding
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-write-forwarding-ams.html
https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-mysql-write-forwarding.html

## Locust neo_saihate

- id, start_at, well_id, quest_id must matched
- quest_id: neo_saihate -> neo_saihate_well -> neo_saihate_quest
- Can find in Avalon: https://development-api.mikoto-gl-dev-as.aktsk.com/avalon#/content/full_master_data/neo_saihate

## GlobalDatabase Failover/Switchover

Event Category
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Events.Messages.html#USER_Events.Messages.cluster

### Switchover
FROM EU to AS
  
AS
```
0181
0182
0184
0070
0006
0071 Completed failover to DB instance:
0185 Global switchover to DB cluster `name` in Region `name` finished.
```

EU
```
0181
0183*10
0182 Old primary DB cluster `name` in Region `name` successfully shut down.
0004 shutdown
0184 New primary DB cluster `name` in Region `name` was successfully promoted.
0006 Restarted
0185 Global switchover to DB cluster `name` in Region `name` finished.
0006 Restarted
```
  

### Failover
FROM AS to EU

EU:
```
0072 : Same AZ failover
0004: Shutdown
0006: Restarted
0071: Completed failover
0238: Completed global failover
```

AS:
```
0004: Shutdown
0240: Started resyncing after failover
0004: Shutdown
Then dead :thinking2:

2hours after..
0020: Recovery of the DB instance has started
0006: Restarted
0004: Shutdown
0006: Restarted
0241: Finished resynchronizing
0021: Recovery of the DB instance is complete.
```



Ref:
https://d1.awsstatic.com/events/reinvent/2020/Deep_dive_on_Global_Database_for_Amazon_Aurora_DAT404.pdf