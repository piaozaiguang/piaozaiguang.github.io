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
一个OpenResty的Kafka Client
下载地址：https://github.com/doujiang24/lua-resty-kafka
lib下的resty/kafka整个拷贝到/usr/local/openresty/{Version}/lualib/下

## nginx.conf

```
cd /usr/local/openresty/nginx/conf
vi nginx.conf
```

```
user  root;
worker_processes  4;

error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

pid        logs/nginx.pid;

events {
    use epoll;
    worker_connections  65535;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    access_log off;
    underscores_in_headers on;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 64k;
    client_body_buffer_size 512k;
    client_max_body_size 50m;

    proxy_buffer_size  32k;
    proxy_buffers      4 512k;
    proxy_busy_buffers_size 1024k;
    proxy_temp_file_write_size 1024k;

    server_tokens   off;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  300;
    tcp_nodelay on;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 16 64k;
    gzip_http_version 1.1;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml application/xml-dtd image/gif image/jpeg image/png image/x-icon image/bmp image/jpg image/x-ms-bmp text/javascript application/x-javascript application/javascript;
    gzip_vary on;

    upstream apps {
        keepalive 80;
        server 192.168.1.20:8080;
        server 192.168.1.21:8080;
    }

    # 引用Kafka Client
    lua_package_path "/usr/local/openresty/1.11.2.3/lualib/kafka/?.lua;;";

    server {
        listen       80;
        server_name  your-domain.com;
        index index.html index.htm index.jsp;
        root html;
        location / {
            proxy_next_upstream http_502 http_504 http_404 error invalid_header;
            proxy_pass http://apps;
            proxy_http_version 1.1;
            proxy_redirect off;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            #proxy_set_header Connection "";
            proxy_set_header X-real-ip $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # 编辑消息体并发送到kafka集群
            log_by_lua '
                local cjson = require "cjson"
                local producer = require "resty.kafka.producer"
                local broker_list = {
                  { host = "192.168.1.150", port = 9092 },
                  { host = "192.168.1.151", port = 9092 },
                  { host = "192.168.1.152", port = 9092 },
                }
                local log_json = {}
                log_json["uri"]=ngx.var.uri
                log_json["args"]=ngx.var.args
                log_json["host"]=ngx.var.host
                log_json["cookie"]=ngx.var.http_cookie
                log_json["method"]=ngx.var.request_method
                log_json["request_body"]=ngx.var.request_body
                log_json["remote_addr"] = ngx.var.remote_addr
                log_json["remote_user"] = ngx.var.remote_user
                log_json["time_local"] = ngx.var.time_local
                log_json["status"] = ngx.var.status
                log_json["body_bytes_sent"] = ngx.var.body_bytes_sent
                log_json["http_referer"] = ngx.var.http_referer
                log_json["http_user_agent"] = ngx.var.http_user_agent
                log_json["http_x_forwarded_for"] = ngx.var.http_x_forwarded_for
                log_json["upstream_response_time"] = ngx.var.upstream_response_time
                log_json["http_current_user"] = ngx.var.upstream_http_x_current_user
                log_json["request_time"] = ngx.var.request_time
                local message = cjson.encode(log_json);
                local bp = producer:new(broker_list, { producer_type = "async" })
                local ok, err = bp:send("accesslog", nil, message)
                if not ok then
                    ngx.log(ngx.ERR, "kafka send err:", err)
                    return
                end
            ';
        }
    }
}
```
