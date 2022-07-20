## 00：文章简介

介绍zabbix_api的使用和批量添加zabbix主机。



## 01：zabbix_api介绍

> 这里使用的zabbix是6.0 LTS版本

官方地址：https://www.zabbix.com/documentation/6.0/en/manual/api

zabbix通过web提供一个接口，允许我们使用post方法传送json-rpc(类json)格式的数据到server。

```bash
请求方法 POST 
请求地址 http://zabbix_server.com/zabbix/api_jsonrpc.php HTTP/1.1
# 这里6.0已经使用 http://zabbix_server.com/api_jsonrpc.php
请求头   Content-Type: application/json-rpc
请求内容 {"jsonrpc":"2.0","method":"apiinfo.version","id":1,"auth":null,"params":{}}
```

## 02：认证用户

认证需要我们发送用户名和密码，这个用户在zabbix web中必须给予api权限

```json
{
	"jsonrpc": "2.0",
	"method": "user.login",
	"params": {
		"user": "Admin",
		"password": "zabbix"
	},
	"id": 1,
	"auth": null
}
```

```
jsonrpc		  - api版本;
method 			- 请求方法;
params 			- 方法携带的参数;
id 					- 原文：an arbitrary identifier of the request， 不是很理解这个id，没有就不行;
auth 				- 用户token令牌
```

返回结果

> 最有用的就是result，里面的值就是接下来使用请求所携带的token

```json
{
	"jsonrpc": "2.0",
	"result": "0424bd59b807674191e7d77572075f33",
	"id": 1
}
```

## 03：获取主机组信息

在zabbix-api的逻辑里，要添加主机，需要主机组id + 模版id，所以需要提前获取这两个内容。

post内容

```json
{
	"jsonrpc": "2.0",
	"method": "hostgroup.get",
	"params": {
		"output": "extend",
		"filter": {
			"name": [#主机组名称, ]
		}
	},
	"auth": user_token,
	"id": 1
}
```

返回结果

```json
{
	'jsonrpc': '2.0',
	'result': [{
		'groupid': '19',
		'name': 'linux98-beijing-M8',
		'internal': '0',
		'flags': '0',
		'uuid': 'd34ca7eda4314c3aa13c1305309d3203'
	}],
	'id': 1
}
```

## 04：获取模版信息

post内容

```json
{
	"jsonrpc": "2.0",
	"method": "template.get",
	"params": {
		"output": "extend",
		"filter": {
			"host": [
				# 模版名称,
			]
		}
	},
	"auth": user_token,
	"id": 1
}
```

返回结果

```json
{
	'jsonrpc': '2.0',
	'result': [{
		'proxy_hostid': '0',
		'host': 'Linux by Zabbix agent',
		'status': '3',
		'lastaccess': '0',
		'ipmi_authtype': '-1',
		'ipmi_privilege': '2',
		'ipmi_username': '',
		'ipmi_password': '',
		'maintenanceid': '0',
		'maintenance_status': '0',
		'maintenance_type': '0',
		'maintenance_from': '0',
		'name': 'Linux by Zabbix agent',
		'flags': '0',
		'templateid': '10001',
		'description': "Official Linux template. Requires agent of Zabbix 3.0.14, 3.4.5 and 4.0.0 or newer.\r\n\r\nKnown Issues:\r\n\r\n  Description: Network discovery. Zabbix agent as of 4.2 doesn't support items such as net.if.status, net.if.speed.\r\n\r\nYou can discuss this template or leave feedback on our forum https://www.zabbix.com/forum/zabbix-suggestions-and-feedback/387225-discussion-thread-for-official-zabbix-template-for-linux\r\n\r\nTemplate tooling version used: 0.41",
		'tls_connect': '1',
		'tls_accept': '1',
		'tls_issuer': '',
		'tls_subject': '',
		'tls_psk_identity': '',
		'tls_psk': '',
		'proxy_address': '',
		'auto_compress': '1',
		'custom_interfaces': '0',
		'uuid': 'f8f7908280354f2abeed07dc788c3747'
	}],
	'id': 1
}
```

## 05：增加主机信息

主机对象字段参考：https://www.zabbix.com/documentation/6.0/en/manual/api/reference/host/object

post内容

```json
{
	"jsonrpc": "2.0",
	"method": "host.create",
	"params": {
		"host": #host_name,
		"name": #visible_name,
		"interfaces": [{
			"type": 1,
			"main": 1,
			"useip": 1,
			"ip": #ipaddr,
			"dns": "",
			"port": "10050"
		}],
		"groups": [{
			"groupid": #host_group_id
		}],
		"templates": [{
			"templateid": #template_id
		}],
		"description": #description
	},
	"auth": #token,
	"id": 1
}
```

返回结果

```json
{
	'jsonrpc': '2.0',
	'result': {
		'hostids': ['10556']
	},
	'id': 1
}
```



## 06：使用python封装

效果说明：

根据excel中的主机清单条目，批量增加到zabbix监控端



依赖模块（只说明需要安装的）

- openpyxl
- requests
- pandas
- tqdm



封装示例

```python
import sys
import json
import pandas
import requests
from tqdm import tqdm


class Zabbix:
    def __init__(self, hostname, username, password):
        self.zabbix_server = "http://{}/api_jsonrpc.php".format(hostname)
        self.username = username
        self.password = password
        self.zabbix_head = {"Content-Type": "application/json-rpc"}
        self.token = self.login()

    def send_post(self, data):
        try:
            result = requests.post(
                url=self.zabbix_server,
                json=data,
                headers=self.zabbix_head
            )
        except Exception as e:
            print(e)
            sys.exit()
        return json.loads(result.text)

    def login(self):
        data = {
            "jsonrpc": "2.0",
            "method": "user.login",
            "params": {
                "username": self.username,
                "password": self.password
            },
            "id": 1
        }
        result = self.send_post(data=data)
        return result.get('result')

    def get_host_groups_id(self, group_name):
        data = {
            "jsonrpc": "2.0",
            "method": "hostgroup.get",
            "params": {
                "output": "extend",
                "filter": {
                    "name": [
                        group_name,
                    ]
                }
            },
            "auth": self.token,
            "id": 1
        }
        result = self.send_post(data=data).get("result")
        if result:
            return result[0].get('groupid')
        print('group {} not found'.format(group_name))
        sys.exit()

    def get_template_id(self, template_name):
        data = {
            "jsonrpc": "2.0",
            "method": "template.get",
            "params": {
                "output": "extend",
                "filter": {
                    "host": [
                        template_name,
                    ]
                }
            },
            "auth": self.token,
            "id": 1
        }
        result = self.send_post(data=data).get("result")
        if result:
            return result[0].get('templateid')
        print('template {} not found'.format(template_name))
        sys.exit()

    def add_host(self, host_name, ipaddr, visible_name, group_name, template_name, description):
        host_group_id = self.get_host_groups_id(group_name)
        template_id = self.get_template_id(template_name)
        data = {
            "jsonrpc": "2.0",
            "method": "host.create",
            "params": {
                "host": host_name,
                "name": visible_name,
                "interfaces": [
                    {
                        "type": 1,
                        "main": 1,
                        "useip": 1,
                        "ip": ipaddr,
                        "dns": "",
                        "port": "10050"
                    }
                ],
                "groups": [
                    {
                        "groupid": host_group_id
                    }
                ],
                "templates": [
                    {
                        "templateid": template_id
                    }
                ],
                "description": description
            },
            "auth": self.token,
            "id": 1
        }
        # 如果执行成功，返回结果中带有result ，如果执行失败，返回结果中带有error
        result = self.send_post(data=data)
        return result


if __name__ == '__main__':
    z = Zabbix("192.168.31.203", "Admin", "zabbix")
    df = pandas.read_excel(r'1.xlsx', sheet_name=1)
    host_list = df.values
    for host in tqdm(host_list):
        description = """
        数据中心: {}，机房：{}，机柜：{}\n应用服务：{}，应用软件：{}\n维护者：{}，日期：{}
        """.format(host[-7], host[-6], host[-5], host[-4],host[-3], host[-2], host[-1])
        z.add_host(
            host_name=host[2],
            ipaddr=host[6],
            visible_name=host[1],
            group_name=host[8],
            template_name=host[9],
            description=description
        )

```

依赖的xlsx文件内容

![image-20220316130832223](https://image.lichunpeng.cn/blog_image/image-20220316130832223.png)


脚本执行效果


![image-20220316131010273](https://image.lichunpeng.cn/blog_image/image-20220316131010273.png)

zabbix-web

![image-20220316131040411](https://image.lichunpeng.cn/blog_image/image-20220316131040411.png)


