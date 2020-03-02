---
title: 基于Keepalived的LVS实验
toc: true
categories:
  - 技术笔记
date: 2020-03-02 21:35:00
tags:
- 高并发
- LVS
- Keepalived
---
### 准备环境
vip：node01 192.168.79.101 

lvs(主)：node01 192.168.79.101  网卡: eth0
lvs(备)：node04 192.168.79.104  网卡: eth0 

nginx1：node02 192.168.79.102   网卡: eth3
nginx2：node03 192.168.79.103   网卡: eth3


### RS中的服务
node02~node03:
1)修改内核：
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore 
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce 
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
2）设置隐藏的vip：
    ifconfig lo:3 192.168.79.110 netmask 255.255.255.255
		
3)在node02 & node03上安装nginx
```
## 解压
tar -xf nginx-1.16.1.tar.gz

## 安装编译工具及库文件 
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

## 配置编译安装
cd nginx-1.16.1
./configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --with-http_ssl_module

make & make install

## 分别修改index.html 加上ip 或内容区分是如台机，方便后续测试
vi /usr/local/nginx/html/index.html

## 启动nginx
./nginx

分别访问检查是否安装成功，如果服务正常开启，但主机访问不了，请检查防火墙
service iptables stop

http://192.168.79.102/
http://192.168.79.103/
```

### 安装ipvsadm & keepalived
在node01 & node04 安装 ipvsadm 和 keepalived

```
## 安装lvs的管理工具ipvsadm
yum install ipvsadm keepalived -y

## 配置
cd  /etc/keepalived/
cp keepalived.conf keepalived.conf.bak
vi keepalived.conf
```

#### keepalived 配置文件内容：
```
! Configuration File for keepalived

## 全局默认配置，发生故障时邮件通知
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.79.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}


## 虚拟路由冗余协议！
vrrp_instance VI_1 {
    ## state MASTER|BACKUP：当前节点在此虚拟路由器上的初始状态；只能有一个是MASTER，余下的都应该为BACKUP；
    ##node4 这里写 BACKUP
    state MASTER
    ## 绑定为当前虚拟路由器使用的物理接口；
    interface eth0
    ## 当前虚拟路由器的惟一标识，范围是0-255
    virtual_router_id 51
    ## 当前主机在此虚拟路径器中的优先级；范围1-254；
    ## node4 这里小点，写 50
    priority 100
    ## vrrp通告的时间间隔；
    advert_int 1
    
    ##  设置认证
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    
    # 定义vip
    virtual_ipaddress {
        192.168.79.110/24 dev eth0 label eth0:1
    }
}

virtual_server 192.168.79.110 80 {
    ## 每隔6秒查看realserver状态
    delay_loop 6
    ## 调度算法改为轮询调度
    lb_algo rr
    # lvs工作模式为DR|NAT|TUN模式
    lb_kind DR
    nat_mask 255.255.255.0
    # 同一IP 的连接50秒内被分配到同一台realserver(测试时改为0)
    persistence_timeout 0
    protocol TCP

    ## 定义realserver
    real_server 192.168.79.102 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            ## 三秒无响应超时
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    ## 定义realserver
    real_server 192.168.79.103 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            ## 三秒无响应超时
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

```


#### 查看keepalived配置帮助文档
```
man 5 keepalived.conf

## 查看virtual_ipaddress 配置说明
virtual_ipaddress {
   <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
   192.168.200.17/24 dev eth1
   192.168.200.18/24 dev eth2 label eth2:1
} 

## brd <IPADDR>: 桥接器地址
## dev <STRING>: 设备
## scope <SCOPE>: 权重值 
## label <LABEL>: 标签
```

#### 启动 & 验证 keepalived
```
## 启动keepalived
service keepalived start

## 检查网卡是否添加成功
ifconfig

## 需要显示信息：
eth0:1    Link encap:Ethernet  HWaddr 00:0C:29:6F:93:B0  
          inet addr:192.168.79.110  Bcast:0.0.0.0  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1


## 检查lvs内核模块的配置：
ipvsadm -ln

## 需要显示信息：
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.79.110:80 rr
  -> 192.168.79.102:80            Route   1      0          0         
  -> 192.168.79.103:80            Route   1      0          0 

```
\* 用浏览器访问http://192.168.79.100 验证结果

### 将DR 的隐藏IP配置写成文件
创建lvsdr.sh，内容如下：
```
#!/bin/sh
VIP=192.168.79.110

# 限制arp请求
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

ifconfig lo:3 $VIP netmask 255.255.255.255
```
