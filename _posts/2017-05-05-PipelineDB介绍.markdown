### 场景
假设有这样的一些需求
* 计数器（比如PV）
* 一些统计业务

### 课题
* 实时性
* 与rawdata解耦
* 数据量
* 负荷
* 程序开发成本

### PipelineDB特性
* 不存原始数据，只存结果 （节省空间）
* 内存计算 （快速）
* SQL查询统计结果
* 实现实时统计（流式计算）

### 概念
![pipelinedb](https://cloud.githubusercontent.com/assets/6111081/25738378/4d1d77b8-31af-11e7-897e-e78030f2136c.png)

### 安装与配置
* http://docs.pipelinedb.com/installation.html
* http://docs.pipelinedb.com/conf.html

### 测试
假设统计每人每天的工作量

#### 建表
```sql
CREATE STREAM daily_stream (
    work_date date,
    worker varchar(20)
);
CREATE CONTINUOUS VIEW daily_view AS
SELECT
    work_date,
    worker,
    COUNT(*) AS cnt
FROM daily_stream
GROUP BY work_date,worker;
```

#### pom.xml
```xml
<!-- PipelineDB是基于postgresql -->
<dependency>
    <groupId>postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>9.2-1002.jdbc4</version>
</dependency>
```

#### 配置数据源
```
pipeline.jdbc.driverClassName=org.postgresql.Driver
pipeline.jdbc.url=jdbc:postgresql://127.0.0.1:5432/test_db
pipeline.jdbc.username=yourname
pipeline.jdbc.password=yourpasswd
```

#### Streaming
```sql
INSERT INTO daily_stream (work_date, worker) VALUES (CURRENT_DATE, 'jhon');
INSERT INTO daily_stream (work_date, worker) VALUES (CURRENT_DATE, 'tom');
INSERT INTO daily_stream (work_date, worker) VALUES (CURRENT_DATE, 'jhon');
INSERT INTO daily_stream (work_date, worker) VALUES (CURRENT_DATE, 'brown');
：
：
```

#### Query
```
select * from daily_view;
 work_date  | worker | cnt 
------------+--------+-----
 2017-05-05 | jhon   |   2
 2017-05-05 | tom    |   1
 2017-05-05 | brown  |   1
(3 rows)
```

#### 查看占用空间
你可以通过`\l+`命令查看它的占用空间，因为它只存结果数据，所以你会发现磁盘占用很少。

### Replication
1. create a role on the primary with "REPLICATION" previledges.
```sh
[user@HOST1 ~]$ pipeline test_db
pipeline (9.5.3)
Type "help" for help.
 
test_db=# CREATE ROLE replicator WITH LOGIN REPLICATION PASSWORD 'xxxxxx';
```
2. add an entry for the standby to the "pg_hba.conf" file.
```sh
host    replication     replicator      192.168.1.2/32         md5
```
3. set a few configuration parameters on the primary by either updating the "pipelinedb.conf" file.
```sh
wal_level = hot_standby
hot_standby = on
max_replication_slots = 1
max_wal_senders = 2 
```
4. create a replication slot for the standby.
```sh
[user@HOST1 ~]$ pipeline test_db
pipeline (9.5.3)
Type "help" for help.
 
test_db=# SELECT * FROM pg_create_physical_replication_slot('replicator_slot');
```
**This is all the setup work we need to do on the primary. Let’s move on to the standby now.**

5. taking a base backup of the primary on the standby.
```sh
[user@HOST1 ~]$ pipeline-basebackup -X stream -D /your/path/pipelinedb/data -h 192.168.1.2 -p 5432 -U replicator
```
6. write a "recovery.conf" in the standby’s data directory.
```sh
standby_mode = 'on'
primary_slot_name = 'replicator_slot'
primary_conninfo = 'host=192.168.1.1 port=5432 user=replicator password=xxxxxx'
recovery_target_timeline = 'latest'
```
7. make sure, connect to the standby and confirm it’s in recovery mode.
```sh
[user@HOST1 ~]$ pipeline test_db
pipeline (9.5.3)
Type "help" for help.
 
test_db=# SELECT pg_is_in_recovery();
 pg_is_in_recovery
-------------------
 t
(1 row)
```
8. retrieve a list of WAL sender processes via the "pg_stat_replication" view.
```sh
[user@HOST1 ~]$ pipeline test_db
pipeline (9.5.3)
Type "help" for help.
 
test_db=# SELECT * FROM pg_stat_replication;
  pid  | usesysid |  usename   | application_name | client_addr  | client_hostname | client_port |         backend_start         | backend_xmin |   state   | sent_location | write_location | flush_location | replay_location | sync_priority | sync_state
-------+----------+------------+------------------+--------------+-----------------+-------------+-------------------------------+--------------+-----------+---------------+----------------+----------------+-----------------+---------------+------------
 22625 |    16384 | replicator | walreceiver      | 192.168.1.2  |                 |       58904 | 2016-07-02 18:55:28.472062+09 |              | streaming | 0/1C8E9D58    | 0/1C8E9D58     | 0/1C8E9D58     | 0/1C8E9A60      |             0 | async
(1 row)
```

### 遇到的坑
* 如果运行一段时间发现内存和CPU变高的话，找到pipelinedb.conf中的autovacuum配置，给off掉。
