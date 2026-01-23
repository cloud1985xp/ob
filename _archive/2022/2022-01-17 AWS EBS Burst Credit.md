---
tags:
  - archive
  - 2022
created: 2022-01-01
updated: 2025-01-23
status: archived
---


# RDS MySQL Got an error reading communication packets
https://aws.amazon.com/tw/premiumsupport/knowledge-center/rds-mysql-communication-packet-error/

# MySQL Metalock
https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html

# EBS
https://aws.amazon.com/tw/blogs/aws/new-burst-balance-metric-for-ec2s-general-purpose-ssd-gp2-volumes/
https://aws.amazon.com/tw/ebs/pricing/
https://docs.aws.amazon.com/zh_tw/AWSEC2/latest/UserGuide/ebs-volume-types.html
https://medium.com/@tdi/burst-balance-ebs-metric-in-aws-a3f3b90261dd
https://stackoverflow.com/questions/50643094/issue-with-ebs-burst-balance-aws
https://aws.amazon.com/tw/blogs/database/understanding-burst-vs-baseline-performance-with-amazon-rds-and-gp2/

https://andyyou.github.io/2021/09/24/aws-ebs-iops/?hmsr=joyk.com&utm_source=joyk.com&utm_medium=referral
https://blog.gslin.org/archives/2016/11/13/6953/aws-%E7%9A%84-general-purpose-ssd-gp2-%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0-burst-io-%E7%9A%84-credit-%E6%95%B8%E5%AD%97%E4%BA%86/


# tcpdump command
https://liferay.dev/blogs/-/blogs/how-to-catch-mysql-sql-with-tcpdump-in-linux
https://huataihuang.gitbooks.io/cloud-atlas/content/network/packet_analysis/tcpdump/tcpdump_avoid_packets_dropped_by_kernel_messages.html


# RDS Tech Team:

```
Hello Aaron,

Thank you for contacting AWS Premium Support. My Name is Saleem and it is my pleasure to assist you with this case.

It was nice chatting with you.

To summarize our conversation, you have contacted us as you noticed aborted connection errors in the aurora clusters.

#Details

- You have upgraded the clusters with name user-xx in the region us-west-1 and there are around 32 clusters.
- After few hours from the cluster upgrade, you started noticing "aborted connections error" in the aurora error logs.

#Sample error :
2022-01-17T12:35:52.720870Z 86495 [Note] Aborted connection 86495 to db: 'ishin_production_user03' user: 'ishin_tw_app' host: '10.3.3.173'

#Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This message is a very common warning from MySQL database engine. It simply indicates either the connection cannot be made to database or the connection between the database server and the client or application is dropped. We normally can find this message in the error log,  it increments the status counter for either aborted_clients or aborted_connects.

---- If a client cannot connect at the application layer, this "Aborted connection" error would increase aborted_connects status variable.
---- If a client is connected but later disconnected improperly or is terminated, this "Aborted connection" error would increase Aborted_clients status variable.

The common causes are :
>> It takes more than connect_timeout seconds to obtain a connect packet.
>> The client/application connected successfully but then disconnected improperly or forcefully. Such as when application disconnected from DB server without having connection properly closed.
>> Database server disconnects idle connections. Such as for connections sleeping more than wait_timeout or interactive_timeout.
>> The client program ended abruptly in the middle of data transfer. Such as a pool network or the data size exchanging with DB server has exceeded the max_allowed_packet.
>> Some application side issue, such as misconfiguration in coding or some libraries or some connection pooling which aborted connection from application side.

* When above are the causes the error includes the actual username and the database name in the error (which we can see that is specified in the error). Since you have not modified any timeout parameters at DB level, it doesn't look like an issue with parameters. Mostly the underlying cause could be with client-side connectivity.

CW_Metrics:
https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(metrics~(~(~'AWS*2fRDS~'AbortedClients~'DBInstanceIdentifier~'user-01-1)~(~'...~'user-01-2)~(~'...~'user-02-1)~(~'...~'user-02-2)~(~'...~'user-03-1)~(~'...~'user-03-2) )~view~'timeSeries~stacked~false~region~'us-west-1~start~'-PT24H~end~'P0D~stat~'Maximum~period~300);query=~'*7bAWS*2fRDS*2cDBInstanceIdentifier*7d*20user-

Further I could see that same error & time patterns matches between cluster instances. And for reader instances seeing these errors at different timelines. Further more the errors indicate only aborted clients.

#Instance user-01-2
2022-01-17T12:35:52.720870Z 86495 [Note] Aborted connection 86495 to db: 'ishin_production_user03' user: 'ishin_tw_app' host: '10.3.3.173' (Got an error reading communication packets)
#Instance user-03-2
2022-01-17T12:35:52.720864Z 86834 [Note] Aborted connection 86834 to db: 'ishin_production_user01' user: 'ishin_tw_app' host: '10.3.3.173' (Got an error reading communication packets)

I have also reviewed if there are any relevant bugs reported for the version Aurora 2.07.7, there are no bugs found.
I have verified the staging clusters where the upgrade is tested "stg-user-01 & stg-user-04" to compare timeout parameters, I did not find any specific timeout configuration on staging clusters.

Resolution : After restarting application processes, the issue is resolved and errors are not seen later.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## Incase if the issue repeats please crosscheck if there are any blocking on Writer instance.
- I have reviewed the Performance Insights graphs and could see that innodb_row_lock_waits and row_lock_time is being reported and it matches with aborted-clients graph. Please check for locking/blocking and meta data locks; in some cases locking / blocking can cause other sessions to wait and timeout.

#### More information on wait time, locked object, etc ####

SELECT r.trx_id AS waiting_trx_id,
r.trx_mysql_thread_id AS waiting_thread,
TIMESTAMPDIFF(SECOND, r.trx_wait_started, CURRENT_TIMESTAMP) AS wait_time,r.trx_query AS waiting_query,
l.lock_table AS waiting_table_lock, b.trx_id AS blocking_trx_id,
b.trx_mysql_thread_id AS blocking_thread, SUBSTRING(p.host, 1,
INSTR(p.host, ':') -1) AS blocking_host, SUBSTRING(p.host,
INSTR(p.host, ':') +1) AS blocking_port, IF(p.command = "Sleep", p.time, 0) AS idle_in_trx,
b.trx_query AS blocking_query
FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS AS w
INNER JOIN INFORMATION_SCHEMA.INNODB_TRX AS b ON b.trx_id = w.blocking_trx_id
INNER JOIN INFORMATION_SCHEMA.INNODB_TRX AS r ON r.trx_id = w.requesting_trx_id
INNER JOIN INFORMATION_SCHEMA.INNODB_LOCKS AS l ON w.requested_lock_id = l.lock_id
LEFT JOIN INFORMATION_SCHEMA.PROCESSLIST AS p ON p.id = b.trx_mysql_thread_id ORDER BY wait_time DESC\G

#### Above queries will not help you in identifying "metatdata locks". For metadata locks; ####
Prior 5.7;
Check your SHOW ENGINE INNODB STATUS and look for prior transactions that hold locks and are still running. Those would be your likely culprits for what is blocking.

> 5.7;
You can query metadata_locks table;
SELECT OBJECT_TYPE, OBJECT_SCHEMA, OBJECT_NAME, LOCK_TYPE, LOCK_STATUS, THREAD_ID, PROCESSLIST_ID, PROCESSLIST_INFO
FROM performance_schema.metadata_locks
INNER JOIN performance_schema.threads ON THREAD_ID = OWNER_THREAD_ID WHERE PROCESSLIST_ID <> CONNECTION_ID();

Useful references:
~~~~~~~~~~~~~~~~
Aborted connections : https://aws.amazon.com/tw/premiumsupport/knowledge-center/rds-mysql-communication-packet-error/
Timeout parameter best practices : https://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-3-parameters-related-to-security-operational-manageability-and-connectivity-timeout/

> connect_timeout: The number of seconds that the MySQLd server waits for a connect packet before responding with Bad handshake.
http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_connect_timeout
> interactive_timeout: Number of seconds the server waits for activity on an interactive connection before closing it
http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_interactive_timeout
> wait_timeout: The number of seconds the server waits for activity on a non-interactive TCP/IP or UNIX File connection before closing it.
https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_wait_timeout
> max_allowed_packet: This value by default is small, to catch large (possibly incorrect) packets. Must be increased if using large BLOB columns or long strings. As big as largest BLOB.
http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_allowed_packet
> net_read_timeout: The number of seconds to wait for more data from a TCP/IP connection before aborting the read.
https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_net_read_timeout
> net_write_timeout: The number of seconds to wait on TCP/IP connections for a block to be written before aborting the write.
https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_net_write_timeout

I hope you find this informative and helpful. For any other questions or concerns, feel free to reach me back through this case. I'll be happy to assist.

We value your feedback. Please share your experience by rating this correspondence using the AWS Support Center link at the end of this correspondence. Each correspondence can also be rated by selecting the stars in top right corner of each correspondence within the AWS Support Center.

Best regards,
Sheik Saleem T.
Amazon Web Services
```


# EC2 Tech. Team:


```
Hello Aaron,

Hope this email finds you well. Kush here from AWS Premium Support. It was a pleasure to assist you on the chat. Kindly find a summary of our discussion.

You reached out to us as you were noticing Ec2 dropping DB connection packets when connected to your RDS Aurora instance.

While we were on chat I have analsysed the tcp dump pcap, but unable to find any anomaly or packet drops, moving further I have verified the output provided by you from where you were noticing packet drops :

>sudo tcpdump -vvv --interface eth0 port 3306 -W 10 -C 100

it shows
```
83019 packets captured
90205 packets received by filter
7149 packets dropped by kernel
```

Here I have checked more about the packet dropped by the kernel and according to Redhat official article I mentioned that "The dropped packets which tcpdump reports are packets which arrived to the system, but which were not delivered to the tcpdump application. The traffic was still received by the system, just not by tcpdump"

-> For more information you can check
[+] https://access.redhat.com/solutions/56608

Moving further I checked the metrics given below using the CloudWatch graphs for the instance 'i-021995a7f613b9b17' but did not find any issues or abnormality at the respective time 2022-01-17 12:00 UTC.

1. NetworkIn and NetworkOut for the instances have been normal was not hitting any performance bottleneck.[1]
2. Instance status checks for the instances did not fail confirming that the software/network configuration of the instances have been fine. [2]
3. System status checks for the instances did not fail confirming that the underlying host is fine on which the instances have been hosted. [3]
4. However I noticed that the CPU utilization of the instance was going to above 95% and you have also confirmed that it was due to your application related process which are consuming the CPU. [4]

Moving further I have checked the EBS volume performance for your root EBS volume 'vol-0af60684ff6882778' allow me to mention the findings :

It is important to understand that the key parameters involved with EBS volumes include IOPS and throughput. IOPS provisioned has some limitations based on the size of the volume as well as burst balance.

Here the burst balance of your EBS volume was getting depleted and reached to 0 starting from 2022-01-17 12:18 UTC after which IOPS were throttled to the baseline IOPS for your volume i.e 192. It means a lot of read and write operations were being done. Depletion of burst balance hinders performance and only GP2 EBS volumes have the concept of burst balance. This goes hand in hand with the size of the volume as well. The volume size determines the baseline performance level of the volume and how quickly it accumulates I/O credits; larger volumes have higher baseline performance levels and accumulate I/O credits faster. Volumes earn I/O credits at the baseline performance rate of 3 IOPS per GiB of volume size. For example, a 200 GiB gp2 volume has a baseline performance of 600 IOPS. However, all volumes with 33.33 GiB and below have a minimum baseline performance of 100 IOPS.

Now since your volume size is 64 GiBs, the baseline for your volume is 192 IOPS. As soon as you start to operate above this 192 IOPS mark, you would be utilizing your burst balance. Once the burst balance depletes completely and reaches 0, your IOPS will be throttled. In your case, the volume went up to a 900 IOPS mainly for the write operation. This caused it to go above the baseline (192 IOPS for 64 GiB) as a result of which burst balance was depleting and reached to 0 at around 2022-01-17 12:18 UTC [5][6]

Looking at the volume performance metrics, it looks like that your workload requires more IOPS, Therefore, I am inclined to recommend you to monitor your application I/O usage and looking at the current usage you may opt to go with gp3 volume type which you can you get minimum of 3000 IOPS with the same volume size as gp2 [7]

I hope this helps.

However if the issue still persist or in case you feel I missed out to address something more to your concern, or if I can otherwise provide any additional assistance with regard to this matter, please do not hesitate to let me know, I’ll be happy to work ahead with you until everything is successfully addressed.

I wish you good day!

References :

[1] NetworkIn and NetworkOut - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(stat~'Maximum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false~metrics~(~(~'AWS*2fEC2~'NetworkIn~'InstanceId~'i-021995a7f613b9b17)~(~'.~'NetworkOut~'.~'.) ));query=~'*7bAWS*2fEC2*2cInstanceId*7d*20i-021995a7f613b9b17

[2] System  status checks - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(stat~'Maximum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false~metrics~(~(~'AWS*2fEC2~'StatusCheckFailed_System~'InstanceId~'i-021995a7f613b9b17) ));query=~'*7bAWS*2fEC2*2cInstanceId*7d*20i-021995a7f613b9b17

[3] Instance status checks - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(stat~'Maximum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false~metrics~(~(~'AWS*2fEC2~'StatusCheckFailed_Instance~'InstanceId~'i-021995a7f613b9b17) ));query=~'*7bAWS*2fEC2*2cInstanceId*7d*20i-021995a7f613b9b17

[4] CPU Utilization - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(stat~'Maximum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false~metrics~(~(~'AWS*2fEC2~'CPUUtilization~'InstanceId~'i-021995a7f613b9b17) ));query=~'*7bAWS*2fEC2*2cInstanceId*7d*20i-021995a7f613b9b17

[5] Burst Balance - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(stat~'Maximum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false~metrics~(~(~'AWS*2fEBS~'BurstBalance~'VolumeId~'vol-0af60684ff6882778) ));query=~'*7bAWS*2fEBS*2cVolumeId*7d*20vol-0af60684ff6882778

[6] Volume Read and Write IOPS - https://us-west-1.console.aws.amazon.com/cloudwatch/home?region=us-west-1#metricsV2:graph=~(metrics~(~(~(expression~'*28m1*2bm2*29*2fPERIOD*28m1*29~label~'Expression1~id~'e1) )~(~'AWS*2fEBS~'VolumeWriteOps~'VolumeId~'vol-0af60684ff6882778~(visible~false~id~'m1))~(~'.~'VolumeReadOps~'.~'.~(visible~false~id~'m2)))~stat~'Sum~period~60~start~'2022-01-17T05*3a30*3a14.000Z~end~'2022-01-18T05*3a30*3a14.000Z~region~'us-west-1~view~'timeSeries~stacked~false);query=~'*7bAWS*2fEBS*2cVolumeId*7d*20vol-0af60684ff6882778

[7] https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html

We value your feedback. Please share your experience by rating this correspondence using the AWS Support Center link at the end of this correspondence. Each correspondence can also be rated by selecting the stars in top right corner of each correspondence within the AWS Support Center.

Best regards,
Kush C.
Amazon Web Services

```