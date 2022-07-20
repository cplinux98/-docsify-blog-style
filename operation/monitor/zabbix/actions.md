## 00：文章简介

介绍zabbix的告警配置和抑制警报。



## 01：告警简介

zabbix平台的告警机制非常灵活，不仅支持分级告警机制，还支持多种样式的告警方式，比如：邮件、微信、钉钉、短信、脚本等等。

zabbix告警流程

item -> Trigger -> action -> media -> 告警通道 -> user



## 02：邮件告警

邮件告警需要有一个邮箱账户，虽然免费，但可能不是很及时。

### 2.1：申请邮箱并配置授权码

![image-20220317145916442](https://image.lichunpeng.cn/blog_image/image-20220317145916442.png)

### 2.2：配置zabbix

> 如果使用ssl/tls，请注意上面的smtp端口

![image-20220317150814042](https://image.lichunpeng.cn/blog_image/image-20220317150814042.png)

配置完成后点击测试

![image-20220317150349858](https://image.lichunpeng.cn/blog_image/image-20220317150349858.png)

查看收件箱

![image-20220317150848613](https://image.lichunpeng.cn/blog_image/image-20220317150848613.png)

### 2.3：用户配置

zabbix需要给用户配置告警媒介之后，才能给用户发送警告

![image-20220317151914800](https://image.lichunpeng.cn/blog_image/image-20220317151914800.png)

### 2.4：tigger action配置

![image-20220317152259382](https://image.lichunpeng.cn/blog_image/image-20220317152259382.png)

![image-20220317152439507](https://image.lichunpeng.cn/blog_image/image-20220317152439507.png)

这里建议是一种故障类型放在一起，不要混合配制，否则容易对故障造成错误判断。

![image-20220317152820917](https://image.lichunpeng.cn/blog_image/image-20220317152820917.png)

上面的step就是告警次数，如果是默认的1-0那么代表一直发送，直到故障恢复为止。

### 2.5：模拟测试

配置完成后，我们对带有该这些触发器的主机进行模拟测试

```
dd if=/dev/zero of=/dev/null bs=1024M count=1000
```

![image-20220317153618900](https://image.lichunpeng.cn/blog_image/image-20220317153618900.png)

![image-20220317153813451](https://image.lichunpeng.cn/blog_image/image-20220317153813451.png)

### 2.6：模板和宏变量参考

对于邮件中的“宏”变量，可以参考官网文档：https://www.zabbix.com/documentation/current/zh/manual/appendix/macros/supported_by_location

我们也可以在media type配置中，在message templates中配置我们的告警模板和恢复模板

恢复模板将bgclolor该为green即可

```html
<table border="1"  bordercolor="black" cellspacing="0px" cellpadding="4px">
<tr >

<td>告警主机</td>
<td bgcolor="red">{HOSTNAME1}</td>
</tr>

<tr>
<td>告警时间</td>
<td>{EVENT.DATE} {EVENT.TIME}</td>
</tr>

<tr>
<td>告警等级</td>
<td>{TRIGGER.SEVERITY}</td>
</tr>

<tr>
<td>告警信息</td>
<td>{TRIGGER.NAME}</td>
</tr>

<tr>
<td>告警项目</td>
<td>{TRIGGER.KEY1}</td>
</tr>
<tr >
<td>问题详情</td>
<td bgcolor="red">{ITEM.NAME}: {ITEM.VALUE}</td>
</tr>
<tr>
<td>当前状态</td>
<td>{TRIGGER.STATUS}: {ITEM.VALUE1}</td>
</tr>
<tr>
<td>事件ID</td>
<td>{EVENT.ID}</td>
</tr>

</table>
```



## 03：脚本邮件告警

对于邮件告警，脚本告警更加灵活，可以根据用户需求自己定义。

### 3.1：脚本存放路径

告警脚本在Zabbix服务器上执行。这些脚本放置于服务器配置文件 **configuration file**中定义的 **AlertScriptsPath**目录下。

```bash
root@zabbix:~# grep AlertSc /etc/zabbix/zabbix_server.conf 
### Option: AlertScriptsPath
# AlertScriptsPath=/usr/lib/zabbix/alertscripts
```

我们取取消脚本路径的注视，并使用默认路径

添加我们的脚本

这里我们使用发送邮件的脚本测试一下

### 3.2：配置脚本

```
apt install libio-socket-ssl-perl libnet-ssleay-perl sendemail -y
```

/usr/lib/zabbix/alertscripts/sendmail.sh

```bash
#!/bin/bash

email_user='linux98_mail@126.com'
email_passwd='xxxxxxxxxx'
smtp_server='smtp.126.com'
email_receive=$1
email_subject=$2
email_message=$3

if [ $# -eq 3 ];then
    /usr/bin/sendemail -f $email_user -t $email_receive -u $email_subject -m $email_message -s $smtp_server -o message-charset=utf-8 -o tls=yes -xu $email_user -xp $email_passwd
fi
```

```bash
chmod +x sendmail.sh
```

### 3.3：脚本测试

命令行测试一下脚本

```
./sendmail.sh linux98_mail@yeah.net "测试脚本发送邮件" "邮件正文"
```

![image-20220317160653712](https://image.lichunpeng.cn/blog_image/image-20220317160653712.png)

### 3.4：配置zabbix-web

新建一个media types

![image-20220317161135061](https://image.lichunpeng.cn/blog_image/image-20220317161135061.png)

在message templates中配置我们的告警模板和恢复模板

> 这里脚本发送html需要前后使用 html body标签包裹

```html
<html>
<body>

# 模板
</body>
</html>
```



![image-20220317161258569](https://image.lichunpeng.cn/blog_image/image-20220317161258569.png)

![image-20220317161329286](https://image.lichunpeng.cn/blog_image/image-20220317161329286.png)

我们把上面配置的tigger action修改为脚本

![image-20220317161612444](https://image.lichunpeng.cn/blog_image/image-20220317161612444.png)

再修改用户的告警类型

![image-20220317162210689](https://image.lichunpeng.cn/blog_image/image-20220317162210689.png)

禁用掉email类型

![image-20220317162223599](https://image.lichunpeng.cn/blog_image/image-20220317162223599.png)

### 3.5：模拟测试

上面完成后，我们对主机使用dd命令进行模拟测试

最后查看邮件

![image-20220317163048881](https://image.lichunpeng.cn/blog_image/image-20220317163048881.png)

## 04：脚本钉钉告警

基于脚本告警的方式，我们配置一个钉钉告警。

### 4.1：创建钉钉群聊

在群设置中点击【智能群助手】

![image-20220317163556331](https://image.lichunpeng.cn/blog_image/image-20220317163556331.png)

### 4.2：配置【自定义】机器人

![image-20220317163904540](https://image.lichunpeng.cn/blog_image/image-20220317163904540.png)

设置机器人名字和安全选项

![image-20220317163749153](https://image.lichunpeng.cn/blog_image/image-20220317163749153.png)

拷贝webhook

![image-20220317163831611](https://image.lichunpeng.cn/blog_image/image-20220317163831611.png)

### 4.3：配置python脚本

> 这里使用简单的示例脚本，需要完善的地方有：日志功能

```python
#!/usr/bin/python3
# coding:utf-8

import requests, json, sys, os, datetime

webhook = "叮叮机器人的webhook"

user = sys.argv[1]
title = sys.argv[2]
text = sys.argv[3]
new_text = "@{}\r\n{}".format(user, text)

data = {
    "msgtype": "markdown",
    "markdown": {
        "title": title,
        "text": new_text
    },
    "at": {
        "atMobiles": [
            user
        ],
        "isAtAll": False
    }
}
headers = {'Content-Type': 'application/json'}
response = requests.post(url=webhook, data=json.dumps(data), headers=headers)
```

命令行测试一下

```
python3 dingding.py 手机号码 "告警信息测试标题" "告警内容"
```

![image-20220317175301867](https://image.lichunpeng.cn/blog_image/image-20220317175301867.png)

测试没问题后，增加dingding告警媒介

### 4.4：配置dingding告警媒介

![image-20220317165655217](https://image.lichunpeng.cn/blog_image/image-20220317165655217.png)

配置告警模板和恢复模板

```
- 告警主机: {HOSTNAME1}
- 告警时间: {EVENT.DATE} {EVENT.TIME}
- 告警等级: {TRIGGER.SEVERITY}
- 告警信息: {TRIGGER.NAME}
- 告警项目: {TRIGGER.KEY1}
- 问题详情: {ITEM.NAME}: {ITEM.VALUE}
- 当前状态: {TRIGGER.STATUS}: {ITEM.VALUE1}
- 事件ID: {EVENT.ID}
```

![image-20220317175516343](https://image.lichunpeng.cn/blog_image/image-20220317175516343.png)

配置用户媒介

![image-20220317175556365](https://image.lichunpeng.cn/blog_image/image-20220317175556365.png)

同样，只开启dingding媒介，关闭其他媒介

![image-20220317175623653](https://image.lichunpeng.cn/blog_image/image-20220317175623653.png)

### 4.5：模拟测试

更新后，对测试主机进行dd模拟测试

```bash
dd if=/dev/zero of=/dev/null bs=1024M count=1000
```

手机端通知显示

![image-20220317195401951](https://image.lichunpeng.cn/blog_image/image-20220317195401951.png ':size=30%')

客户端查看信息

![image-20220317195452356](https://image.lichunpeng.cn/blog_image/image-20220317195452356.png ':size=30%')

## 05：分阶告警

对于运维团队来说，肯定不会只把告警信息发送给1个人，当然每个人处理的故障级别也不同，这里我们演示一下分阶级告警。

效果：

- 发生告警后，立即通知给A同事
  - A同事处理了，问题结束
  - A同事未处理
    - 继续告警，发送给同事B
      - 同事B仍未处理
        - 发送给A、B的领导
          - 。。。

就这样逐层上报，直到问题被解决。

### 5.1：创建用户

分别创建3个用户，来模拟A、B、领导

> 接收告警的用户需要是超级管理员super admin role权限才能正常接收

| 名字     | 职务     | 邮箱                                    |
| -------- | -------- | --------------------------------------- |
| zhangsan | 运维A    | z3@linux98.com                          |
| lisi     | 运维B    | l4@linux98.com |
| wangwu   | 运维经理 | w5@linux98.com|

### 5.2：调整tigger action

![image-20220317202604626](https://image.lichunpeng.cn/blog_image/image-20220317202604626.png)

如果需要说明后续步骤的故障通知原因，可以自定义标题，说明

```
这里面的Default operation step duration，与步骤里的step duration的作用是：
发生问题的多少时间后，发送给下一个用户，也就是给用户预留的操作时间或响应时间

step 后面为0代表循环操作
	先给1发，1不处理
		给1和2发
			1和2都没处理
				给1 + 2 + 3一起发
					都没处理，一直给123发
```

![image-20220317215004951](https://image.lichunpeng.cn/blog_image/image-20220317215004951.png)

![image-20220317215957064](https://image.lichunpeng.cn/blog_image/image-20220317215957064.png)

## 06：维护告警

当有主机告警了，我们进入到维护期间，此时zabbix不应该继续告警。


>  创建之前我们先触发之前的告警

![image-20220317223522167](https://image.lichunpeng.cn/blog_image/image-20220317223522167.png)

Configuration -> Maintenance -> Create maintenance period

!> 这里的维护类型，可以选择收集数据和不收集数据

- 收集数据，适用于发生故障后维护主机，在你维护机器的过程中，告警会自己消失，并发送故障修复通知；
- 不收集数据，适用于批量维护设备，并提前配置。

![image-20220317224317733](https://image.lichunpeng.cn/blog_image/image-20220317224317733.png)

配置完成后，等待一下就会生效了，此时主机的告警就消失了，并且不会再发送警告邮件了。

等我们维护完成，删除掉该维护状态即可。




