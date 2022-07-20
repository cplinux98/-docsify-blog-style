## 00：文章简介

介绍zabbix-server中自定义trigger。



## 01：介绍trigger

zabbix中的item监控项仅负责收集数据，对收到的数据一般需要进行简单的分析，然后根据特定的指标进行评判，trigger触发器能够对数据进行监控，并发出告警或执行动作。

每一个trigger仅能管理一个item，但同一个item可以使用多个trigger，以实现对不同级别的处理，也就是分级告警机制。

trigger由表达式构成，对数据的阈值进行对比，超出转为"PROBLEM"，小于则为"OK"。



## 02：创建trigger

Configuration -> Hosts -> 任意主机 -> tiggers -> Create tigger

![image-20220316185017795](https://image.lichunpeng.cn/blog_image/image-20220316185017795.png)

![image-20220316184842621](https://image.lichunpeng.cn/blog_image/image-20220316184842621.png)

配置完成后，再查看graph，就会有一条线了，这就是阈值，超过就会发出警告

```bash
# 模拟cpu繁忙
dd if=/dev/zero of=/dev/null bs=1024M count=1000
```



![image-20220316185237094](https://image.lichunpeng.cn/blog_image/image-20220316185237094.png)

在Monitoring中可以看到警告和历史警告

![image-20220316185605627](https://image.lichunpeng.cn/blog_image/image-20220316185605627.png)

我们多配置几个触发器，对cpu1分钟负载分成3个阶段

- 大于0.65
  - 1分钟负载有点高了
- 大于0.85
  - 1分钟负载高了
- 大于0.95
  - 1分钟负载太高了

> 可以点击触发器，然后clone，这样可以快速创建

![image-20220316190319566](https://image.lichunpeng.cn/blog_image/image-20220316190319566.png)

## 03：创建模板

对于上面的各种自定义来说，1台主机还好，可是主机多了就力不从心了，这时我们可以对每种业务类型定制模板。

> 这里我创建一个空模板，不链接任何模板，自定义几个监控项和触发器

创建一个空主机组

![image-20220316192029428](https://image.lichunpeng.cn/blog_image/image-20220316192029428.png)



![image-20220316192117372](https://image.lichunpeng.cn/blog_image/image-20220316192117372.png)

添加item

点击模板 -> items -> Create item

![image-20220316191445877](https://image.lichunpeng.cn/blog_image/image-20220316191445877.png)

![image-20220316191543764](https://image.lichunpeng.cn/blog_image/image-20220316191543764.png)

创建graph

点击模板 -> graphs -> Create graph

![image-20220316191643688](https://image.lichunpeng.cn/blog_image/image-20220316191643688.png)

创建tigger

点击模板 -> tiggers -> Create tigger

![image-20220316192343265](https://image.lichunpeng.cn/blog_image/image-20220316192343265.png)

![image-20220316192423427](https://image.lichunpeng.cn/blog_image/image-20220316192423427.png)

![image-20220316192534553](https://image.lichunpeng.cn/blog_image/image-20220316192534553.png)

最后效果

![image-20220316192704102](https://image.lichunpeng.cn/blog_image/image-20220316192704102.png)

创建一个主机，添加时选择该模板

!> 该模板种使用了自定义监控项，创建主机时应该配置好

![image-20220316193030688](https://image.lichunpeng.cn/blog_image/image-20220316193030688.png)

等待主机上线，查看主机的item、trigger、graph

![image-20220316203250568](https://image.lichunpeng.cn/blog_image/image-20220316203250568.png)

![image-20220316203309462](https://image.lichunpeng.cn/blog_image/image-20220316203309462.png)

## 04：创建检测TCP链接数模版

创建检测脚本      /etc/zabbix/zabbix_agentd.d/tcp_status.sh

```bash
#!/bin/bash
user_status=$1

tcp_status(){
  TCP_STAT=$1
  TCP_STAT_VALUE=$(ss -ant | awk 'NR>1 {++s[$1]} END {for(k in s) print k,s[k]}' | grep "$TCP_STAT" | cut -d ' ' -f2)
  if [ -z $TCP_STAT_VALUE ];then
      TCP_STAT_VALUE=0
  fi
  echo $TCP_STAT_VALUE
}

tcp_status $user_status
```

配置自定义参数   /etc/zabbix/zabbix_agentd.d/tcp_status.conf

```
UserParameter=tcp_status[*],/bin/bash /etc/zabbix/zabbix_agentd.d/tcp_status.sh "$1"
```

重启agent

```
systemctl restart zabbix-agent.service
```

使用zabbix_get测试

```
zabbix_get -s 192.168.31.21 -k tcp_status[ESTAB]
```

创建模板

Configuration -> Templates -> Create template

配置监控项

![image-20220316220408387](https://image.lichunpeng.cn/blog_image/image-20220316220408387.png)

按照上图所示配置好11种状态(使用clone)

```
CLOSE_WAIT
CLOSED
CLOSING
ESTAB
FIN-WAIT-1
FIN-WAIT-2
LAST-ACK
LISTEN
SYN-RECV
SYN-SENT
TIME-WAIT
```

![image-20220316221443911](https://image.lichunpeng.cn/blog_image/image-20220316221443911.png)

配置一个estab连接数大于5500就告警的配置

![image-20220316221630730](https://image.lichunpeng.cn/blog_image/image-20220316221630730.png)

将所有item添加到新的graph中

![image-20220316221733304](https://image.lichunpeng.cn/blog_image/image-20220316221733304.png)

对一个主机应用该模板测试

![image-20220316221835685](https://image.lichunpeng.cn/blog_image/image-20220316221835685.png)


