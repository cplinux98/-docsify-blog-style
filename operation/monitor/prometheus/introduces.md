## 00：文章简介

介绍prometheus的基本信息。



## 01：Prometheus简介

Prometheus是CNCF中的“毕业”监控产品。它基于go语言开发，使用的是TSDB时序数据库。

官网：https://prometheus.io/

Prometheus以出色的可视化、集成度高、多客户端而出名，尤其是在监控kubernetes容器集群场景中。

Prometheus典型特征：从每个Target中拉取pull数据，所以优势在于：

- 集中控制，有利于将配置集成在Prometheus Server中；
- 目标明确，目标在于收集Target上预先完成聚合的聚合型数据。

Prometheus需要对每个环境的每个指标进行数据获取，并按照用户需要的方式提供给用户。



## 02：metric简介

我们要监控数据，首先得有数据，数据应该以直接或间接的方式暴露出来，方式如下

- 硬件本身的记录，以log或内存属性的方式存在
- 应用业务的接口，主动暴露软件自身运行状态，比如redis info、status等
- 相关的信息采集工具，方便收集数据，基于系统级别的命令

这些数据在长时间的运行过程中，都是以固定的“属性指标”来描述他们，也就是metric。

对于metric数据指标，我们需要持续性、周期性的方式来采集，通常采集方式有：

- exporters
  - 类似于agnet，帮助监控端生成http接口的metric
- instrumentation
  - 监控端自带的状态暴露
- pushgateway
  - 不稳定的监控端发送数据给它，prometheus从它上面获取稳定的数据

无论哪种方式，server端都是pull模式工作的。

## 03：时序数据库

对于我们采集到的“监控指标”，不是一次性使用的，我们需要将数据存储起来，并与采集的时间相关联，方便我们知道这些数据是“什么时候发生的”。我们将这种监控场景的数据称为：时间序列数据 TS（Time series data），存储这种数据的数据库，就是时序数据库(TSDB Time series database)。

时序数据库是用来存储时序列(time-series)数据并以时间（点或区间）建立索引的软件。

一般时序数据库都具备下面的特点：

- 数据结构简单，某一度量指标在某一时间点只会有一个值，没有复杂的结构和关系
- 数据量大，可以存储大量的数据指标

## 04：Prometheus数据特点

- Prometheus采用独特的指标格式，我们称之为Prometheus格式

- 支持多维度标签，每个独立的标签组合都代表一个独立的时间序列

- prometheus内部支持多种数据的聚合、切割、切片

- prometheus支持双精度浮点型数据存储

## 05：Prometheus架构介绍

![image-20220318125636654](https://image.lichunpeng.cn/blog_image/image-20220318125636654.png)

工作流程为：

1. server定期从jobs中拉取metrics发过来的metrics
2. server在本地存储收集到的metrics，并运行定义好的alert.rules，并记录或向alertmanager推送警报
3. Alertmanager根据配置文件，对接收到的警报进行去重分组，根据路由配置，向对应主机发出告警
4. 集成Grafana或其他API作为图形界面，用于可视化收集的数据

## 06：Prometheus基本部件

| 组件               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| Prometheus Server  | 主服务，接收外部http请求，收集、存储、查询数据               |
| Client Library     | 客户端库，为需要监控的服务生成相应的metrics并暴露给Prometheus Server，当Prometheus server  来pull时，直接返回实时状态的metrics |
| Push Gateway       | 数据收集，用于短期的jobs，主要用于服务层面     被监控端向push gateway发送metrics，prometheus server从push gateway中收集metrics |
| Exporters          | 节点agent，一般用于对设备采集，并以http方式暴露metrics       |
| Alertmanager       | 报警管理器，从对alerts进行去重，分组，并路由到对应的接收方式，以高效的向用户发送告警信息 |
| Data Visualization | Prometheus Web UI ，Grafana，为Prometheus Server内建支持     |
| Service Discovery  | 动态发现待监控的Traget，从而完成监控配置的重要组件，在容器化环境中尤为有用，为Prometheus Server内建支持。 |
| PromQL             | 对数据操作的接口，用于自身web-ui展示或给Grafana数据展示提供接口 |

## 07：Prometheus数据模型

Prometheus仅用于以“键值”形式存储时序式的聚合数据，它并不支持存储文本信息

![image-20220318135235102](https://image.lichunpeng.cn/blog_image/image-20220318135235102.png)

- 其中的键称为指标metric，它通常意味着cpu速率、内存使用率、分区空闲比例等

- 同一指标可能会适配多个目标或设备，因此它使用“标签”作为元数据，从而为metric添加更多信息描述纬度

- 这些标签还可以作为过滤器进行指标过滤和聚合运算

## 08：Prometheus指标类型

prometheus使用4种方法来描述监视的指标

- Counter
  - 计数器，用于保存单调递增的数据，例如站点访问次数，不能为负值，不能减少，但可以重置为0
- Gauge
  - 用于存储有着起伏特征的指标数据，例如内存空闲大小
- Histogram
  - 直方图
- Summary
  - 摘要

## 09：Job和Instance

![image-20220318140549727](https://image.lichunpeng.cn/blog_image/image-20220318140549727.png)

- target
  - 一组运行业务并带有向外暴露metric接口，包含了暴露的http url 和端口的采集点，称之为target
- Instance
  - 能够接收Prometheus Server数据Scrape操作的每个网络端点（endpoint），即为一个Instance实例
- Job
  - 通常具有类似功能的Instance的集合称为一个Job，例如一个MySQL主从复制集群中的所有MySQL进程

如果和zabbix相比较的话，job相当于Host groups，instance相当于hosts，target相当于`hostip:agent_type:port`



## 10：PromQL

Prometheus提供了内置的数据查询语言PromQL，支持用户进行实时的数据查询及聚合操作。

PromQL支持两种向量，并内置提供了一组用于数据处理的函数。

- 即时向量
  - 最近一次的时间戳上跟踪的数据指标
- 时间范围向量
  - 指定时间范围内的所有时间戳上的数据指标

![image-20220318140945749](https://image.lichunpeng.cn/blog_image/image-20220318140945749.png)

## 11：Alerts

抓取到异常值后，Prometheus支持通过“告警”机制向用户发送反馈或警示，以触发用户能够及时采取应对措施；

Prometheus Server仅负责生成告警指示，具体的告警行为由另一个独立的应用程序AlertManager负责。

!> 对于生产环境，prometheus server和alertmanager都要做到高可用，否则告警组件坏了，告警就没作用了

- 告警指示由Prometheus server基于用户提供的“告警规则”周期性计算生成；
- alertmanager接收到Prometheus Server发来的告警指示后，基于自定义的告警路由向告警接收人发送信息


