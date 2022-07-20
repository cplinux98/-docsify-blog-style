## 00：文章简介

介绍了LVS中的NAT模型的原理与特性，以及模拟了企业中使用场景。


## 01：LVS 的NAT模型概念

通过修改请求报文的目标IP地址，而后根据调度算法挑选出一台RS节点进行转发。

## 02：NAT访问原理

1. 当用户请求到达DS，此时请求的数据报文会先到内核空间的PREROUTING链。此时报文的源IP为CIP，目标IP为VIP

2. PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链；
3. IPVS对比数据包请求的服务是否为集群服务，如果是，则通过调度算法挑选1台后端RS服务器，并修改数据包的目标IP为RS的IP，然后将数据包发至POSTROUTING链。此时报文的源IP为CIP，目标IP为RIP；
4. POSTROUTING链通过选路，将数据包通过DS的DIP发送给RS；
5. RS发现目标为自己的IP，则交给应用程序处理，然后构建响应报文发回给DS。此时报文的源IP为RIP，目标IP为CIP；
6. DS在响应客户端前，会将源IP地址修改为VIP地址，然后响应给客户端。此时报文的源IP为VIP，目标IP为CIP。


| 名称 | 详细名称          | 描述                               |
| ---- | ----------------- | ---------------------------------- |
| DS   | Director Server   | 目标服务器，即负载均衡器LVS        |
| RS   | Real Server       | 真实应用服务器，即后端服务器       |
| CIP  | Client IPV        | 客户端请求IP                       |
| VIP  | Virtual IP        | 直接面向用户的IP地址，通常为公网IP |
| DIP  | Diector Server IP | 用于与后端RIP通信的IP地址          |
| RIP  | Real Server IP    | 后端真实服务器的IP地址             |


## 03：NAT特性

1. RS必须使用私网地址，并需要将网关指向DS；
2. RIP和DIP必须同一网段；
3. NAT模型支持端口映射；
4. RS可以使用任意操作系统，如linux、windows等；
5. 请求和响应报文都要经过DS，高负载场景中，DS容易成为瓶颈。


## 04：NAT模型实践

### 4.1：架构规划
![](https://gitee.com/lichunpeng12/linux98_images/raw/master/images/20210518142246.png)

### 4.2：地址规划

![](https://gitee.com/lichunpeng12/linux98_images/raw/master/images/Xnip2021-10-27_11-53-20.png)

### 4.3：环境准备

01-web展示环境

```bash
#web01、web02
[root@web01 ~]# cat /etc/nginx/conf.d/web.linux98.com.conf 
server {
        listen 80;
        server_name web.linux98.com;

        location / {
                root /code_8080;
                index index.html;
        }
}
[root@web01 ~]# cat /code_8080/index.html 
welcome to web01
[root@web02 ~]# cat /code_8080/index.html 
welcome to web02

#proxy01
[root@proxy01 ~]# cat /etc/nginx/conf.d/proxy_web.linux98.com.conf 
upstream web {
        server 172.16.1.7:80;
        server 172.16.1.8:80;
}

server {
        listen 80;
        server_name web.linux98.com;

        location / {
                proxy_pass http://web;
                include proxy_params;
        }
}

[root@proxy01 ~]# nginx -t
[root@proxy01 ~]# systemctl restart nginx

#部署proxy02节点
[root@proxy01 ~]# scp -rp /etc/nginx root@172.16.1.6:/etc/
[root@proxy02 ~]# yum install nginx
[root@proxy02 ~]# systemctl enable nginx
[root@proxy02 ~]# nginx -t
[root@proxy02 ~]# systemctl start nginx
```

02-client节点配置

```bash
#client 
[root@client ~]# ifdown eth1
#注释掉eth0的网关和dns，使client相当于外网主机
[root@client ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
#GATEWAY=10.0.0.1
#DNS1=223.5.5.5
#DNS2=114.114.114.114
#重启eth0
[root@client ~]# ifdown eth0 && ifup eth0

[root@client ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
```

03-route节点配置

```bash
#route网络配置、开启forward转发
查看route两个网卡地址
[root@route ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO="static"
DEFROUTE=yes
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=10.0.0.200
GATEWAY=10.0.0.1
PREFIX=24
DNS1=223.5.5.5
[root@route ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=172.16.1.200
PREFIX=24

#这里的网卡配置文件一定要简洁，不需要uuid和其他问题，不然会导致后期client访问时好时坏

[root@route ~]# ip a
[root@route ~]# ip a| grep "inet " 
    inet 127.0.0.1/8 scope host lo
    inet 10.0.0.200/24 brd 10.0.0.255 scope global noprefixroute eth0
    inet 172.16.1.200/24 brd 172.16.1.255 scope global noprefixroute eth1
[root@route ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        0.0.0.0         UG    100    0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
172.16.1.0      0.0.0.0         255.255.255.0   U     101    0        0 eth1
[root@route ~]# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf 
[root@route ~]# sysctl -p
net.ipv4.ip_forward = 1

#通常在企业内部想要向外映射网站需要在防火墙上开一个端口映射，这里的route也需要开启端口映射
[root@route ~]# iptables -t nat -I PREROUTING -p tcp --dport 80 -d 10.0.0.200 -j DNAT --to 172.16.1.100:80
```

04-proxy01、proxy02节点配置

```bash
#proxy01、proxy02
#网关指向DS的ip地址
#使用带有10和172网段的主机进行跳转ssh

[root@proxy01 ~]# eth1
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=172.16.1.5
GATEWAY=172.16.1.3
PREFIX=24
[root@proxy01 ~]# ifdown eth0
[root@proxy01 ~]# ifdown eth1 &&&& ifup eth1

[root@proxy02 ~]# eth1
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=172.16.1.6
GATEWAY=172.16.1.3
PREFIX=24

#关闭eth0，使之成为内网环境
[root@proxy02 ~]# ifdown eth0
[root@proxy02 ~]# ifdown eth1 >> ifup eth1
```

05-lvs-master节点配置

```bash
#lvs-master节点  (使用带有10和172网段的主机进行跳转ssh)
#安装lvs工具
[root@lvs-master ~]# yum -y install ipvsadm
#配置网络地址
#增加eth1子接口虚拟网卡
[root@lvs-master ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1:1
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=eth1:1
DEVICE=eth1:1
ONBOOT=yes
IPADDR=172.16.1.100
PREFIX=24
[root@lvs-master ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth1
增加
GATEWAY=172.16.1.200
#重启网卡后，关闭eth0
[root@lvs-master ~]# systemctl restart network
[root@lvs-master ~]# ifdown eth0
[root@lvs-master ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.16.1.200    0.0.0.0         UG    101    0        0 eth1
172.16.1.0      0.0.0.0         255.255.255.0   U     101    0        0 eth1
172.16.1.0      0.0.0.0         255.255.255.0   U     101    0        0 eth1

#开启forward转发
#如果不开启forward转发，RS（proxy节点）发送给DS的数据包会被lvs丢弃，因为目标ip地址是10网段，不是lvs的ip地址
[root@lvs-master ~]# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf 
[root@lvs-master ~]# sysctl -p
net.ipv4.ip_forward = 1
```

### 4.4：配置LVS规则

```bash
[root@lvs-master ~]# ipvsadm -A -t 172.16.1.100:80 -s rr
[root@lvs-master ~]# ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.5:80 -m
[root@lvs-master ~]# ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.6:80 -m
[root@lvs-master ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.16.1.100:80 rr
  -> 172.16.1.5:80                Masq    1      0          0         
  -> 172.16.1.6:80                Masq    1      0          0
```

### 4.5：使用client访问测试

```bash
[root@client ~]# curl -Hhost:web.linux98.com http://10.0.0.200
welcome to web01
[root@client ~]# curl -Hhost:web.linux98.com http://10.0.0.200
welcome to web02
```

### 4.6：访问过程注解
![](https://gitee.com/lichunpeng12/linux98_images/raw/master/images/20210520103503.png)


### 4.7：访问过程中源IP与目标IP的变化
![](https://gitee.com/lichunpeng12/linux98_images/raw/master/images/20210520103504.png)


## 05：模型中存在的疑点

### 5.1：客户端client是与哪个节点进行的握手？
```bash
#1、对web节点进行限速处理
#web01、web02节点
[root@web02 ~]# vim /etc/nginx/conf.d/web.linux98.com.conf 
server {
        listen 80;
        server_name web.linux98.com;
        limit_rate 100k;

        location / {
                root /code_8080;
                index index.html;
        }
}
#创建一个大文件
[root@web02 ~]# dd if=/dev/zero of=/code_8080/bigdate bs=300M count=1
[root@web02 ~]# nginx -t
[root@web02 ~]# systemctl reload nginx

#2、使用client客户端进行下载
#client节点
[root@client ~]# echo "10.0.0.200 web.linux98.com" >> /etc/hosts
[root@client ~]# wget web.linux98.com/bigdata

#3、在route、lvs、proxy、web节点上查看ESTABLISHED
[root@route ~]# netstat -an | grep -i established | grep 80

[root@lvs-master ~]# netstat -an | grep -i established | grep 80

[root@proxy01 ~]# netstat -an | grep -i established | grep 80
tcp        0      0 172.16.1.5:53080        172.16.1.7:80           ESTABLISHED
tcp        0      0 172.16.1.5:80           10.0.0.10:34550         ESTABLISHED

[root@proxy02 ~]# netstat -an | grep -i established | grep 80
tcp        0      0 172.16.1.6:22           172.16.1.61:48980       ESTABLISHED

[root@web01 ~]# netstat -an | grep -i established | grep 80
tcp        0      0 172.16.1.7:80           172.16.1.5:53080        ESTABLISHED

[root@web02 ~]# netstat -an | grep -i established | grep 80

#由上面的连接查看得知，client是与proxy进行的三次握手
```

### 5.2：proxy已经是轮询的了，怎么能看出客户端访问通过lvs也是轮询的？

```bash
#1、关闭proxy01节点的nginx代理
[root@proxy01 ~]# systemctl stop nginx

#2、使用客户端访问web.linux98.com测试
[root@client ~]# curl web.linux98.com
curl: (7) Failed connect to web.linux98.com:80; Connection refused
[root@client ~]# curl web.linux98.com
welcome to web01
[root@client ~]# curl web.linux98.com
curl: (7) Failed connect to web.linux98.com:80; Connection refused
[root@client ~]# curl web.linux98.com
welcome to web02
[root@client ~]# curl web.linux98.com

#可以看出访问失败时是调度到proxy01节点，再次访问调度到proxy02节点，proxy02节点又进行了2次web集群轮询
```

### 5.3：LVS中的RS节点故障，没有自动健康检查机制吗？
>
LVS本身没有健康检查机制，需要LVS配合keepalived来做集群，这样就有健康检查机制了。
