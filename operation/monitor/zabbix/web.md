## 00：文章简介

介绍zabbix监控web场景。



## 01：web场景监控

zabbix提供了一种专门用于web场景的监控 web scenarios，可以基于web项目的url页面的访问状态来做多方面的服务监控，比如网站可用性、响应速度、请求处理时间等。

### 1.1：模拟web场景

找一个主机安装好nginx，测试能否访问

![image-20220318120652236](https://image.lichunpeng.cn/blog_image/image-20220318120652236.png)

### 1.2：配置web监控

在zabbix-web的Configuration -> Hosts中找到该主机，并点击该主机的web选项，Create web scenarios

![image-20220318120923299](https://image.lichunpeng.cn/blog_image/image-20220318120923299.png)

![image-20220318121125128](https://image.lichunpeng.cn/blog_image/image-20220318121125128.png)

查看监控状态

Monitoring -> Hosts -> 目标主机 -> web

![image-20220318121403224](https://image.lichunpeng.cn/blog_image/image-20220318121403224.png)

### 1.3：故障模拟

修改nginx主页文件名称，使其无法正常访问

```
mv /var/www/html/index.nginx-debian.html{,.bak}
```

![image-20220318121601523](https://image.lichunpeng.cn/blog_image/image-20220318121601523.png)


