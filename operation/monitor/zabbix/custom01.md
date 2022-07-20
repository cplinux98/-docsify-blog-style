## 00：文章简介

介绍zabbix-server中items的原理以及如何自定义item、graph和dashboard。



## 01：认识items

items就是zabbix被监控端的监控项目，通常由key-value结构

查看items-key，Configuration -> Hosts -> 任意主机->items

![image-20220316161420809](https://image.lichunpeng.cn/blog_image/image-20220316161420809.png)

查看items-value，Monitoring -> Latest data

![image-20220316161539830](https://image.lichunpeng.cn/blog_image/image-20220316161539830.png)



## 02：自定义tems

官方文档介绍

```
https://www.zabbix.com/documentation/current/zh/manual/config/items/userparameters
```

根据官网的介绍，我们以CPU负载为例子，测试1、5、15分钟的负载的命令

```bash
uptime | awk -F ": |, " '{print $4}'
uptime | awk -F ": |, " '{print $5}'
uptime | awk -F ": |, " '{print $6}'
```

再使用官方推荐格式写入到文件中

```
Format: UserParameter=<key>,<shell command>
```

/etc/zabbix/zabbix_agentd.d/cpu_load.conf 

```
UserParameter=cpu.load1,uptime | awk -F ": |, " '{print $4}'
UserParameter=cpu.load5,uptime | awk -F ": |, " '{print $5}'
UserParameter=cpu.load15,uptime | awk -F ": |, " '{print $6}'
```

重启服务

```
systemctl restart zabbix-agent.service
```

在zabbix-server中使用zabbix_get命令检测效果（来自zabbix-get包）

```bash
root@zabbix:~# zabbix_get -s 192.168.31.200 -p 10050 -k "cpu.load1"
0.00
root@zabbix:~# zabbix_get -s 192.168.31.200 -p 10050 -k "cpu.load5"
0.04
root@zabbix:~# zabbix_get -s 192.168.31.200 -p 10050 -k "cpu.load15"
0.06
```

在web端创建item条目

Configuration -> Hosts -> 任意主机 -> Create item

![image-20220316164046199](https://image.lichunpeng.cn/blog_image/image-20220316164046199.png)

增加之前可以点test进行测试

![image-20220316164125592](https://image.lichunpeng.cn/blog_image/image-20220316164125592.png)

增加完成后，就可以在Monitoring -> Latest data中选择主机查看效果了

![image-20220316165308349](https://image.lichunpeng.cn/blog_image/image-20220316165308349.png)



## 03：自定义Graph

Configuration -> Hosts -> 任意主机 -> Graphs -> Create graph

![image-20220316165415477](https://image.lichunpeng.cn/blog_image/image-20220316165415477.png)

配置完成后，可以点击Preview查看样式和效果

> 这里图示中如果配置中文，需要添加中文字体

![image-20220316165513903](https://image.lichunpeng.cn/blog_image/image-20220316165513903.png)

![image-20220316174522116](https://image.lichunpeng.cn/blog_image/image-20220316174522116.png)

## 04：自定义dashboard

Monitoring -> Dashboard -> All dashboards -> Create dashboard

![image-20220316175735952](https://image.lichunpeng.cn/blog_image/image-20220316175735952.png)

用的最多的类型就是Graph、Map、Url、clock、Action log等

最后效果

![image-20220316183423372](https://image.lichunpeng.cn/blog_image/image-20220316183423372.png)


