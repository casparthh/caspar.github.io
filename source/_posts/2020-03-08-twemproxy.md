---
title: Twemproxy(Nutcracker)
toc: true
categories:
  - 技术笔记
date: 2020-03-08 00:45:46
tags:
  - Redis
---
Twtter 开源的一个 Redis 和 Memcache 代理服务器，主要用于管理 Redis 和 Memcached 集群。
<!--more-->
### 编译
git :https://github.com/twitter/twemproxy
下载源码
安装依赖工具
`$ yum install automake libtool`

添加阿里云epel repository
`$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo`
`$ yum clean all`

查找新版本的autoconf
`$ yum search autoconf`
`$ yum install autoconf268`

 从源码编译
`$ makereconf -fvi`
`$ ./configure`
`$ make`

### 安装
编译完成后进入script目录下，可以找到nutcracker.init文件，这是个脚本文件可以打开看下。
里面有指定了
a. chkconfig
b. 执行程序： 
    prog="nutcracker"
c. 配置文件nutcracker.yml： 
  OPTIONS="-d -c /etc/nutcracker/nutcracker.yml"

接下来需要做的就是：
1.将这个脚本文件复制到 /etc/init.d/
`$ cp nutcracker.init /etc/init.d/nutcracker`
`$ chmod +x /etc/init.d/nutcracker`

2.回到编译目录下找到nutcracker.yml配置文件并复制到/tec/nutcracker/目录下
`$ mkdir /etc/nutcracker`
`$ cp ./conf/nutcracker.* /etc/nutcracker/`

3.在编译文件的src目录下找到程序nutcracker,并复制到/usr/bin/下
`$ cp ./src/nutcracker /usr/bin/`


4.修改nutcracker
文件中有配置好了几个参考模版。我们只需保留任意一个，然后修改services部分指向redis master就行。
```  
alpha:
  listen: 127.0.0.1:22121
  hash: fnv1a_64
  distribution: ketama
  auto_eject_hosts: true
  redis: true
  server_retry_timeout: 2000
  server_failure_limit: 1
  servers:
   - vm1:6379:1
   - vm3:6379:1
```
auto_eject_host: 当连接一个server失败次数超过server_failure_limit值时，是否把这个server驱逐出集群，默认是false  
server_retry_timeout:单位毫秒，当auto_eject_host打开后，重试被临时驱逐的server之前的等待时间    
server_failure_limit: 当auto_eject_host打开后，驱逐一个server之前重试次数  

### 运行测试
启动服务  
`$ service nutcracker start`  

连接到代理程序并测试
`$ redis-cli -p 22121`
`set pass mypassword`
`set k1 mytestk1`
set 多个key 后再分别到多台master上get 验证，由于算法原因，可能会连续很多key存到了同一台redis, 所以多set一些不同的key验证。

其缺点：
不支持Redis的事务操作。    
不支持针对多个值的操作，比如取sets的子交并补等。  
  

