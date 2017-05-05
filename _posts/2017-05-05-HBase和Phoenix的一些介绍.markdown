### HBase
> [Apache HBase](http://hbase.apache.org/) is an open-source, distributed, versioned, non-relational database modeled after Google's [Bigtable: A Distributed Storage System for Structured Data](http://research.google.com/archive/bigtable.html).

#### Install & Configuration
* http://hbase.apache.org/book.html

#### 逻辑数据结构
![hbase logical view](https://cloud.githubusercontent.com/assets/6111081/25735160/c99150a8-319b-11e7-8d33-7b68bb6d0c28.png)
#### 逻辑数据结构
![hbase data model](https://cloud.githubusercontent.com/assets/6111081/25735158/c329699e-319b-11e7-8552-783af9d8470e.jpg)

* Table
* RowKey：数据的主键，分布，快速查找
* Column Family：列的组合，列族独立分割存取
* Column：属于某一个列族，可以随时加
* Version Number：版本（时间戳）
* Value(Cell)：值（Byte array）

#### 基本操作
* Create
```sh
$ create 'test', 'cf'
0 row(s) in 3.4170 seconds
```
* Put
```sh
$ put 'test', 'row1', 'cf:a', 'value1'
0 row(s) in 0.1540 seconds
```
* Get
```sh
$ get 'test', 'row1'
COLUMN                CELL
 cf:a                 timestamp=1407130286968, value=value1
1 row(s) in 0.0110 seconds
```
* Scan
```sh
$ scan 'test'
ROW                   COLUMN+CELL
 row1                 column=cf:a, timestamp=1407130286968, value=value1
 row2                 column=cf:b, timestamp=1407130286997, value=value2
 row3                 column=cf:c, timestamp=1407130287007, value=value3
 row4                 column=cf:d, timestamp=1407130287015, value=value4
```
* Delete
```sh
$ delete 'test', 'row1', 'cf:a'
$ deleteall 'test', 'row1'
```
* No update
* And more...

#### 架构
![hbasearch](https://cloud.githubusercontent.com/assets/6111081/25735191/21191b08-319c-11e7-8082-27848f81ec10.jpg)

* HMaster
  * 管理HRegionServer，健康检查，负载均衡
  * 分配HRegion，Rebalance
  * DDL操作
  * namespace，table的元数据
  * ACL
* HRegionServer
  * 管理HRegion
  * 读写HDFS
  * Client请求
* Zookeeper
  * HBase集群的元数据
  * HBase集群的状态
  * HMaster的failover
  
#### 访问过程
![first time request from client](https://cloud.githubusercontent.com/assets/6111081/25735144/b9bd010e-319b-11e7-8534-d715f2d1ae1c.png)
#### 读/写缓存(Region server)
![region server cache](https://cloud.githubusercontent.com/assets/6111081/25735194/298d8af8-319c-11e7-8e06-7a8eae5c2d52.png)
#### Rowkey介绍
* HBase Query
  * 使用Rowkey Get数据 (< 1ms)
  * Rowkey范围的Scan(StartRowkey, EndRowkey)
  * Full scan (二级索引)
  * 数据是Ordered by rowkey
* HRegion
  * 按Rowkey分成多份HRegion，每个HRegion都有Startkey和Endkey
  * 顺序递增的Rowkey导致Region热点 → [link](https://sematext.com/blog/2012/04/09/hbasewd-avoid-regionserver-hotspotting-despite-writing-records-with-sequential-keys/)
* Rowkey的设计
  * 避免单点热点，负载均衡
  * 有没有Rowkey范围查询的需求

#### 二级索引(Secondary indexes)
* Rowkey的不足
  * 一般HBase查询是通过Rowkey，那如果复合查询条件呢？能把复合查询条件都拼到Rowkey中？考虑业务的多样化，这不太现实！
* 二级索引？
  * 其实HBase并不像RDBMS那样提供索引，一般采用第三方solution, 比如典型的索引列和Rowkey的映射方式:
![secondary index design](https://cloud.githubusercontent.com/assets/6111081/25735198/33ea196c-319c-11e7-8b9c-843496508d9f.png)

### Phoenix
> A relational database layer for Apache HBase → [Official site](https://phoenix.apache.org/)
* SQL Engine
* JDBC Driver
* Secondary indexes
* Paging
* Subqueries
* Join
* Grouping
* More...

#### Install
1. Add the phoenix-[version]-server.jar to the classpath of all HBase region server and master.
2. Restart HBase.
3. Add the phoenix-[version]-client.jar to the classpath of any Phoenix client.

#### SQL Client
![squirrel](https://cloud.githubusercontent.com/assets/6111081/25736147/b2f8d2ec-31a2-11e7-8b5f-13f59bcdab0b.png)

#### 基本操作
```sql
CREATE TABLE IF NOT EXISTS api_error_log (
    channel_id CHAR(4) NOT NULL,
    content_id VARCHAR(100) NOT NULL,
    create_time TIMESTAMP NOT NULL,
    api_url VARCHAR,
    response_body VARCHAR,
    CONSTRAINT api_error_log_pk PRIMARY KEY (channel_id,content_id,create_time)
) SALT_BUCKETS = 48;
  
CREATE INDEX my_idx ON api_error_log(api_url DESC) INCLUDE (response_body) SALT_BUCKETS=48;

SELECT * FROM TEST LIMIT 1000 OFFSET 100;
  
UPSERT INTO TEST(NAME,ID) VALUES('foo',123);
  
DELETE FROM TEST WHERE ID=123;
```

#### Global indexes
read heavy

#### Local indexes
write heavy

hbase-site.xml
```xml
<property>
  <name>hbase.regionserver.wal.codec</name>
  <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
</property>
```

### 应用在公司项目中的经验

#### Why HBase?
* 不断增长的数据量 - Scale out

#### Why Phoenix?
* SQL/JDBC
* 低延迟，实时性
* 从RDBMS平移过来

#### 存哪些数据？
* 采用Hybrid方案，热数据还是存在RDBMS中，历史记录等冷数据存到HBase中

### 踩过的坑
* salt (bucket)
  * 单点热点问题，建表时指定bucket数来预拆分
  * 但，不按顺序存
* upsert带来的程序复杂度
  * insert + update
  * 没有主键冲突的概念
* 二级索引
  * 索引列的顺序，在where条件中至少用上索引第一列，比如只用索引的第二列就不走索引
  * include常用字段
* 分页查询
  * limit，offset貌似是按Rowkey，和预想的结果不一致
  * bucket也有嫌疑(调查中)
* HDFS的master重启带来的影响
  * hbase.rootdir没有指定HDFS Cluster，写成特定的namenode，所以namenode重启时候HBase集群受到影响
* 数据迁移
  * 写程序？
  * Bulkload（基于MapReduce）
  * Sqoop？
