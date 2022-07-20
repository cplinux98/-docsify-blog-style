## 00：文章简介

由于zabbix设备的磁盘容量不足，需要对磁盘进行扩容，为了防止扩容时出现意外，先备份好数据库和相关配置文件，以防万一！


<!-- more -->



## 01：备份zabbix模板文件

配置-->模板-->选择要备份的模板-->导出
导出的为.xml格式文件
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353330713642.jpg)
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353330778754.jpg)


## 02：备份zabbix模板文件

配置-->主机-->选择名称-->导出
导出文件也为xml格式

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353331029600.jpg)

## 03：备份数据库配置文件

zabbix数据库中有许多表分为“存放数据的表”和“配置表”。
存放数据的表：
alerts 
auditlog 
events 
historyhistory_log 
history_str 
history_str_sync 
history_sync 
history_text 
history_uint 
history_uint_sync 
node_cksum 
proxy_dhistory 
proxy_history
service_alarms
services_times 
trends 
trends_uint
备份配置表数据
#使用--ignore-table跳过不需要备份的表
```bash
]# mysqldump -uroot -p123456 --databases zabbix  --ignore-table=zabbix.alerts --ignore-table=zabbix.auditlog  --ignore-table=zabbix.events  --ignore-table=zabbix.history  --ignore-table=zabbix.history_log --ignore-table=zabbix.str --ignore-table=zabbix.str_sync  --ignore-table=zabbix.sync  --ignore-table=zabbix.text --ignore-table=zabbix.uint  --ignore-table=zabbix.uint_sync  --ignore-table=zabbix.node_cksum  --ignore-table=zabbix.proxy_dhistory --ignore-table=zabbix.proxy_history --ignore-table=zabbix.service_alarms --ignore-table=zabbix.services_times --ignore-table=zabbix.trends --ignore-table=zabbix.trends_uint > zabbix_config.sql
```