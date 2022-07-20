## 00：文章简介

介绍zabbix监控相关知识。



## 01：数据监控流程

正常来说，一次完整的监控至少需要有下面几种流程

![image-20220316134255303](https://image.lichunpeng.cn/blog_image/image-20220316134255303.png)

```
数据采集：
	根据业务的特性，采取不同的方式，进行一些针对性的数据进行采集
数据整理：
  对上报后的数据源进行收集、清洗、整理
实时分析：
  对某些重要的核心的业务数据，进行实时分析
离线分析：
  对普通的数据、非紧急的业务数据进行存储，后续进行相应的分析
结果输出：
  将实时分析和离线分析后的数据结果展示出来，供决策参考
问题决策：
  根据业务情况，人工或自动对输出的结果进行分析，并判定下一步的行动（告警修复），同时将其决策记录保存下来，以便为后续决策提供依据
```

> 核心目标就是 维持网站的可用性



## 02：Linux系统监控指标

图片来源：

https://www.brendangregg.com/linuxperf.html

https://www.brendangregg.com/Perf/linux_observability_tools.png

![image-20220316135414826](https://image.lichunpeng.cn/blog_image/image-20220316135414826.png)

![image-20220316135754011](https://image.lichunpeng.cn/blog_image/image-20220316135754011.png)

类似的图片在源网站还有很多，可以根据需要查找



## 03：Zabbix基本架构

![image-20220316134139270](https://image.lichunpeng.cn/blog_image/image-20220316134139270.png)



zabbix的服务端和客户端的监控机制有主动模式和被动模式（相对于客户端）

- 主动模式
  - 客户端请求server获取主动的监控项列表，并主动返回监控项内的数据给server/proxy
- 被动模式（默认）
  - server向客户端请求获取监控项数据，agnet返回数据

## 04：硬件要求

![image-20220316145543713](https://image.lichunpeng.cn/blog_image/image-20220316145543713.png)


