## 00：文章简介

介绍安装部署zabbix及使用ansible配置zabbix-agent。



## 01：Zabbix安装

环境介绍

- 主机系统：Ubuntu 20.04 LTS
- 软件版本：Zabbix 6.0 LTS
- 浏览器版本：Google Chrome 版本 99.0.4844.74（正式版本） (x86_64)
- SSH软件版本：SecureCRT Version 9.1.1



需要提前安装好mysql-server

```bash
apt install mysql-server -y
```

剩余参考地址 https://www.zabbix.com/cn/download?zabbix=6.0&os_distribution=ubuntu&os_version=20.04_focal&db=mysql&ws=nginx

安装完成后，初始默认密码为：Admin zabbix



初始化

初始化界面很简单，这里只说明需要注意的点

```
time zone: Asia/Shanghai
```



## 02：创建组和用户

Administration -> User groups -> Create user group

![image-20220315210628583](https://image.lichunpeng.cn/blog_image/image-20220315210628583.png)

Administration -> Users  -> Create user

![image-20220315210837321](https://image.lichunpeng.cn/blog_image/image-20220315210837321.png)

permissions

> 需要详细配置角色的，可以自行配置角色权限，这里统一使用管理员权限了

![image-20220315210910139](https://image.lichunpeng.cn/blog_image/image-20220315210910139.png)

## 03：添加主机组和主机

Configuration -> Host groups -> Create host group

![image-20220315211350203](https://image.lichunpeng.cn/blog_image/image-20220315211350203.png)





Monitoring -> Hosts -> Create host

> 这里只以ansible主机为示例，其余注意参照添加

![image-20220315220056250](https://image.lichunpeng.cn/blog_image/image-20220315220056250.png)

需要注意的就是主机名和ip地址要与真实主机一致，描述信息尽量详细，其他默认即可。

## 04：手动安装客户端

```bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
apt update
apt install zabbix-agent
```

如果有很多节点需要安装zabbix-agent，可以考虑为其在本地创建仓库代理

```bash
# cat /etc/apt/sources.list.d/zabbix.list 
deb https://repo.zabbix.com/zabbix/6.0/ubuntu focal main
deb-src https://repo.zabbix.com/zabbix/6.0/ubuntu focal main
```

配置客户端

/etc/zabbix/zabbix_agentd.conf 

```
Server=192.168.31.203				# 服务器地址
ListenPort=10050						# 监听端口，使用默认
Hostname=control  					# 对应添加主机里的Host name，如果不配置，默认使用系统主机名
ServerActive=192.168.31.203 # 主动提交服务器地址
```

启动服务

```bash
systemctl restart zabbix-agent.service 
systemctl enable zabbix-agent.service 
```

检查

```bash
# 端口
ss -nutlp | grep zabbix
# 日志
/var/log/zabbix/zabbix_agentd.log
```

等待一会儿后，在web上control的图标就变色了，此时就可以查看信息了

## 05：批量安装客户端

```
└── zabbix-agent
    ├── files
    │   └── zabbix-release_6.0-1+ubuntu20.04_all.deb
    ├── tasks
    │   └── main.yml
    └── templates
        └── zabbix_agentd.conf.j2
```

task/main.yml

```yaml
- name: Install Zabbix Source Deb (1/2)
  copy:
    src: "zabbix-release_6.0-1+ubuntu20.04_all.deb"
    dest: "/tmp/zabbix-release_6.0-1+ubuntu20.04_all.deb"

- name: Install Zabbix Source Deb (2/2)
  apt:
    deb: "/tmp/zabbix-release_6.0-1+ubuntu20.04_all.deb"

- name: Install Zabbix Agent
  apt:
    name: zabbix-agent
    state: present
    update_cache: yes

- name: Config Zabbix Agent
  template:
    src: "zabbix_agentd.conf.j2"
    dest: "/etc/zabbix/zabbix_agentd.conf"
    owner: "root"
    group: "root"
    mode: "0644"

- name: Restart Zabbix Agent Service
  systemd:
    name: zabbix-agent
    state: started
    enabled: yes
```

templates/zabbix_agentd.conf.j2

```jinja2
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server={{ zabbix_server_ip }}
ListenPort=10050
Hostname={{ ansible_hostname }}
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

运行playbook

```
- hosts:
    - dns
    - nexus
    - gitlab
    - jenkins
    - harbor
    - lb
    - k8s-master
    - k8s-node
  roles:
    - zabbix-agent
```



这里可以使用自动发现，或者导入文件方式快速添加多台主机

## 06：批量导入主机

文件方式：Configuration -> Hosts -> 勾选主机 -> Export

api方式：https://linux98.com/#/operation/soft/zabbix/zabbix_api

## 07：zabbix-java-gateway安装

对于java应用，zabbix可以通过JMX的方式来进行主机的监控，开启JMX后能监控tomcat服务运行时出现的故障，以及监视内存、CPU等使用情况。

Zabbix-server部署zabbix-java-gateway

```
apt install zabbix-java-gateway -y
```

/etc/zabbix/zabbix_java_gateway.conf

> 参考地址https://www.zabbix.com/documentation/current/en/manual/appendix/config/zabbix_java

```
LISTEN_IP="0.0.0.0"
LISTEN_PORT=10052
PID_FILE="/var/run/zabbix/zabbix_java_gateway.pid"
START_POLLERS=50
TIMEOUT=30
```

```bash
systemctl restart zabbix-java-gateway
systemctl enable zabbix-java-gateway
 netstat -nutlp | grep java
```

/etc/zabbix/zabbix_server.conf 追加下面的配置

```
JavaGateway=192.168.31.203
JavaGatewayPort=10052
StartJavaPollers=20  # 这里的数量要比zabbix-java-gateway 配置中的START_POLLERS 小
```

重启检查

```bash
root@zabbix:~# systemctl restart zabbix-server.service 
root@zabbix:~# ps aux | grep 'java poller'
```



添加主机

```
template选择 Templates/Applications -> Apache Tomcat by JMX -> Generic Java JMX
```

![image-20220316155135123](https://image.lichunpeng.cn/blog_image/image-20220316155135123.png)



## 08：java客户端安装

安装一个tomcat应用并启动

```bash
apt install tomcat9
# 检查是否运行
curl localhost:8080
```

添加java启动选项

/usr/share/tomcat9/bin/catalina.sh 

找到 export JDK_JAVA_OPTIONS ，下面内容放在它前面

> 这里只做简单演示，详情查看https://www.zabbix.com/documentation/3.4/zh/manual/config/items/itemtypes/jmx_monitoring

```
JDK_JAVA_OPTIONS="$JDK_JAVA_OPTIONS -Djava.rmi.server.hostname=192.168.31.200 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
```

重启服务并检查端口

```bash
systemctl restart tomcat9
ss -nutlp | grep java
```

等待一会儿后，检查web中的主机情况

## 09：配置中文语言包（可选）

使用zabbix过程中，可能配置中文的图表，需要显示中文

安装中文包

```
apt-get install language-pack-zh*
```

/etc/environment 

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
# 语言和编码配置
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"
```

重设语言配置`dpkg-reconfigure locales`

![image-20220316170830685](https://image.lichunpeng.cn/blog_image/image-20220316170830685.png)

![image-20220316170857551](https://image.lichunpeng.cn/blog_image/image-20220316170857551.png)

设置中文字体

```
windows在 c:\windows\Fonts中选择字体
Mac在App 字体册中导出字体
```

上传字体到/usr/share/zabbix/assets/fonts/目录下

```
unlink graphfont.ttf
ln -s 新字体 graphfont.ttf
```

最后重启zabbix相关服务

```
systemctl restart zabbix-server zabbix-agent nginx
systemctl restart php7.4-fpm.service
```

web配置

```
User settings -> Profile -> Language
```


