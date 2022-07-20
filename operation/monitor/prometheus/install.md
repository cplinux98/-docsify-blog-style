## 00：文章简介

介绍prometheus的安装部署。



## 01：Prometheus部署

### 1.1：直接运行二进制文件

安装包获取：https://prometheus.io/download/

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.34.0/prometheus-2.34.0.linux-amd64.tar.gz

tar -xf prometheus-2.34.0.linux-amd64.tar.gz -C /usr/local/
cd /usr/local/
ln -s prometheus-2.34.0.linux-amd64/ prometheus
cd prometheus
./prometheus # 前台启动，并以当前目录的prometheus.yml作为配置文件运行
```

自定义prometheus程序配置

```bash
mkdir {bin,cfg}
mv prometheus.yml cfg/
mv prometheus promtool bin/
```

配置环境变量

/etc/profile.d/prometheus.sh 

```
#!/bin/bash
export PROMETHEUS_HOME=/usr/local/prometheus
export PATH=$PATH:${PROMETHEUS_HOME}/bin
```

使其生效

```
source /etc/profile.d/prometheus.sh 
chmod +x /etc/profile.d/prometheus.sh 
```

配置服务文件

/usr/lib/systemd/system/prometheus.service

```
[Unit]
Description=prometheus server project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/prometheus/bin/prometheus --config.file=/usr/local/prometheus/cfg/prometheus.yml --storage.tsdb.path=/usr/local/prometheus/data
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

设置服务开机自启动

```
systemctl enable prometheus
systemctl start prometheus
```

### 1.2：apt源安装

在apt源中也可以安装prometheus系列软件包，但是版本不是最新的。

```bash
root@control:~# apt list |grep prometheus

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

golang-github-grpc-ecosystem-go-grpc-prometheus-dev/focal 1.2.0+git20191002.6af20e3-2 all
golang-github-mcuadros-go-gin-prometheus-dev/focal 0.1.0+git20190723.c7374e9-2 all
golang-github-prometheus-alertmanager-dev/focal-updates 0.15.3+ds-3ubuntu1 all
golang-github-prometheus-client-golang-dev/focal 0.9.2-0ubuntu3 amd64
golang-github-prometheus-client-model-dev/focal 0.0.2+git20171117.99fa1f4-1 all
golang-github-prometheus-common-dev/focal 0.7.0-1 all
golang-prometheus-client-dev/focal 0.9.2-0ubuntu3 amd64
libnet-prometheus-perl/focal 0.07-1 all
prometheus-alertmanager/focal-updates 0.15.3+ds-3ubuntu1 amd64
prometheus-apache-exporter/focal 0.7.0+ds-1 amd64
prometheus-bind-exporter/focal 0.3.0+ds-1 amd64
prometheus-bird-exporter/focal 1.2.2-1 amd64
prometheus-blackbox-exporter/focal 0.13.0+ds-2 amd64
prometheus-hacluster-exporter/focal 0.4.0-2 amd64
prometheus-haproxy-exporter/focal 0.9.0+git20180917+ds-1 amd64
prometheus-ipmi-exporter/focal 1.1.0+ds-1 amd64
prometheus-mailexporter/focal 1.0-2 amd64
prometheus-mongodb-exporter/focal 1.0.0+git20180522.e755a44-1 amd64
prometheus-mysqld-exporter/focal 0.11.0+ds-1 amd64
prometheus-nginx-exporter/focal 0.5.0+ds-2 amd64
prometheus-nginx-vts-exporter/focal 0.10.3+git20180501.43b4556+ds-2 amd64
prometheus-node-exporter-collectors/focal,now 0+git20200110.fc91c86-1 all [installed,automatic]
prometheus-node-exporter/focal,now 0.18.1+ds-2 amd64 [installed]
prometheus-pgbouncer-exporter/focal 1.7-1 all
prometheus-postfix-exporter/focal 0.2.0-2 amd64
prometheus-postgres-exporter/focal 0.8.0+ds-1 amd64
prometheus-process-exporter/focal 0.4.0+ds-1 amd64
prometheus-pushgateway/focal 1.0.0+ds-1 amd64
prometheus-snmp-exporter/focal 0.16.1+ds-1build1 amd64
prometheus-sql-exporter/focal 0.2.0.ds-6 amd64
prometheus-squid-exporter/focal 1.8.2+ds-1 amd64
prometheus-trafficserver-exporter/focal 0.3.2-1 all
prometheus-varnish-exporter/focal 1.5.2-1 amd64
prometheus-xmpp-alerts/focal 0.4.2-1 all
prometheus/focal,now 2.15.2+ds-2 amd64 [installed]
python3-django-prometheus/focal 2.0.0-1 all
python3-prometheus-client/focal 0.7.1-1.1 all
ruby-prometheus-client-mmap/focal 0.10.0-1build1 amd64
```

对于服务启动文件，有需要的可以参考apt安装后的服务启动文件

```bash
/lib/systemd/system/prometheus.service
/lib/systemd/system/prometheus-node-exporter.service
```



## 02：添加node_exporter监控端

在被监控的主机上安装node_exporter

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xf node_exporter-1.3.1.linux-amd64.tar.gz -C /usr/local
cd /usr/local
ln -s node_exporter-1.3.1.linux-amd64 node_exporter
```

自定义node_exporter程序配置

```bash
cd node_exporter
mkdir bin
mv node_exporter bin/
```

配置环境变量

/etc/profile.d/node_exporter.sh 

```
#!/bin/bash
export NODE_EXPORTER_HOME=/usr/local/node_exporter
export PATH=$PATH:${NODE_EXPORTER_HOME}/bin
```

使其生效

```
source /etc/profile.d/node_exporter.sh 
chmod +x /etc/profile.d/node_exporter.sh 
```

配置服务文件

/usr/lib/systemd/system/node_exporter.service

```
[Unit]
Description=prometheus node exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/node_exporter/bin/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

设置服务开机自启动

```
systemctl enable node_exporter
systemctl start node_exporter
```

访问主机ip:9100/metrics查看效果

```
curl 192.168.31.11:9100/metrics
```

## 03：添加Job

这里的job类似于zabbix中的Host Groups

/usr/local/prometheus/cfg/prometheus.yml 

```yaml
... # 只配置scrape_config的内容
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "nodes"          # 组名
    metrics_path: '/metrics'   # 配置metrics的uri
    static_configs:
      - targets:               # groups中的host，列表形式
        - '192.168.31.11:9100'
```

重启prometheus服务查看

![image-20220318151356025](https://image.lichunpeng.cn/blog_image/image-20220318151356025.png)

## 04：grafana部署

### 4.1：安装软件

官方地址：https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1

```
apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_8.4.4_amd64.deb
dpkg -i grafana-enterprise_8.4.4_amd64.deb

# 安装插件
grafana-cli plugins list-remote
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins ls

# 启动服务
systemctl start grafana-server.service
systemctl status grafana-server.service
systemctl enable grafana-server.service 

# 检查端口
ss -nutlp | grep "grafana"
```

使用浏览器访问3000端口，默认用户名密码admin/admin

### 4.2：添加数据源

Configuration -> datasources -> add your data source -> 选择prometheus

```
URL: http://localhost:9090/
```

### 4.3：导入模板

"+"号图标 -> Import

官方模板站点：https://grafana.com/grafana/dashboards/

这里我选用了8919和1860模板

```
https://grafana.com/grafana/dashboards/8919
https://grafana.com/dashboards/1860
```

![image-20220318153706026](https://image.lichunpeng.cn/blog_image/image-20220318153706026.png)

### 4.4：查看效果

!> 如果没有图形，请查看主机和你浏览器的时间是否准确

![image-20220318153850171](https://image.lichunpeng.cn/blog_image/image-20220318153850171.png)




