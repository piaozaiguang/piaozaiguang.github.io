## 场景
Nginx的Access log有时候对我们来说很有价值，但是当成log文件一直放在服务器上的话价值并不大，而且还占磁盘空间。

其实通过log做各种分析，统计等方案挺多，我想在这里介绍一下利用OpenResty+Kafka+ELK搭建日志系统。

## OpenResty
国人杰作，[OpenResty](http://openresty.org/cn/)

可以在Nginx之上利用Lua语言编写模块儿。

[Nginx](http://nginx.org)：诞生在俄罗斯，一个高性能的HTTP和反向代理服务器

[Lua](http://www.lua.org)：诞生在巴西，脚本语言，易于嵌入到其他语言，因此很多人叫它胶水语言

有人说OpenResty是来自[金砖国家](https://baike.baidu.com/item/金砖国家)

## Kafka
[Kafka](http://kafka.apache.org)是一个诞生在LinkedIn的高性能分布式消息系统，用Scala语言编写，目前已经贡献给Apache基金会。

## ELK
[ELK](https://www.elastic.co/cn/)是Elasticsearch+Logstash+Kibana的缩写

Elasticsearch是基于Lucene的分布式全文搜索引擎

Logstash：收集log，处理log的工具，可以利用Logstash直接把log文件内容存到Elasticsearch中，但是为了进一步编辑log内容，本文没有采用Logstash

Kibana：Elasticsearch的可视化平台

## 思路

- 用OpenResty替代原来的Nginx
- 用Lua脚本改造Accesslog打印，把log发送到Kafka集群中
- Kafka的Consumer Group异步处理log队列
- 基于log内容编辑数据并存到Elasticsearch中

## 安装OpenResty
下载最新的源码：https://openresty.org/cn/download.html
```sh
# 解压
tar -xzvf openresty-VERSION.tar.gz
```
```sh
cd openresty-{VERSION}
./configure # 可以通过--prefix=/your/directory 参数指定安装目录
# 默认安装到/usr/local/openresty目录

# 如果报Missing OpenSSL错误执行下面命令
yum install readline-devel pcre-devel openssl-devel gcc curl
```
```sh
# 编译
make

# 安装
make install
```

## lua-resty-kafka
一个
## lua-resty
