原文：http://blog.51cto.com/hudamao/2102895

## git clone报错提示

```sh
git clone https://github.com/xxxx.git
Initialized empty Git repository in /root/xxxx/.git/
error:  while accessing https://github.com/xxxx.git/info/refs
fatal: HTTP request failed
```

## 解决办法

```sh
yum update -y nss curl libcurl
```
