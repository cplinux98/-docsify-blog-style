## 00：文章简介

介绍prometheus的如何部署在k8s集群中。



版本信息：

- kubernetes: v1.23.4
- kubeadm: v1.23.4
- kube-prometheus: 0.10
- prometheus: 2.32.1



## 01：kube-prometheus项目介绍

coreos对prometheus在k8s运行开源了一套管理接口: kube-prometheus

目前项目已经转移到：https://github.com/prometheus-operator/kube-prometheus

这个组织中开源了2个关于prometheus运行在k8s中的项目：

- Prometheus Operator
  - 通过用户自定义的CRD资源及Controller实现
  - 监听这些自定义资源的变化，并根据变化自动完成Prometheus Server的配置及工作
- Kube-prometheus
  - 即包含了Operator，又包含了Prometheus相关组件的部署及常用的Prometheus自定义监控
  - 包括下面的组件
    - The Prometheus Operator：创建CRD自定义的资源对象
    - Highly available Prometheus：创建高可用的Prometheus
    - Highly available Alertmanager：创建高可用的告警组件
    - Prometheus node-exporter：创建主机的监控组件
    - Prometheus Adapter for Kubernetes Metrics APIs：创建自定义监控的指标工具（例如可以通过nginx的request来进行应用的自动伸缩）
    - kube-state-metrics：监控k8s相关资源对象的状态指标
    - Grafana：进行图像展示

> Prometheus Operator架构图 

引用地址：https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/Documentation/user-guides/images/architecture.png

![image-20220322185817921](https://image.lichunpeng.cn/blog_image/image-20220322185817921.png)

## 02：kube-prometheus部署

### 2.1：下载并解压

使用git-clone或下载release版本

```bash
wget https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v0.10.0.tar.gz
tar -xf kube-prometheus-0.10.0.tar.gz
```

### 2.2：提前下载镜像

对于kube-prometheus中的部分镜像，由于某些原因是无法下载的，需要通过阿里云对镜像进行代理，可以参考：https://linux98.com/#/operation/soft/docker/imageproxy

对于网络没问题的同学，可以使用下面的脚本下载并导入本地harbor仓库。

```bash
cd /kube-prometheus-0.10.0/manifests
```

download.sh

```bash
#!/bin/bash
#login
expect -c "
spawn docker login harbor.linux98.com
        expect {
                \"*Username:*\" {send \"cpli\r\"; exp_continue}
                \"*Password:*\" {send \"A12345678a\r\"; exp_continue}
        } "

# download and upload
images=$(cat *.yaml | grep image: | tr -d ' ' | sort | uniq | cut -d':' -f2,3)

for i in ${images}
do
    docker pull $i
    newName=$(echo "$i" | awk -F'/' '{print $NF}')
    docker tag $i harbor.linux98.com/kube-prometheus/$newName
    docker rmi $i
    docker push harbor.linux98.com/kube-prometheus/$newName
done

# replace image path

for j in ${images}
do
    filename=$(grep "$j" *.yaml| cut -d":" -f1)
    for k in ${filename}
    do
        newName=$(echo "$j" | awk -F'/' '{print $NF}')
        # echo "filename: $k, image: $newName"
        sed -i "s#$j#harbor.linux98.com/kube-prometheus/$newName#g" $k
    done
done
```

运行上面的镜像，会批量下载需要的image、上传至本地harbor、替换文件中的image路径

### 2.3：应用资源清单

镜像下载完成后，就可以应用清单了

```bash
kubectl apply --server-side -f manifests/setup
kubectl apply -f manifests/
```

最后效果

![image-20220321212320107](https://image.lichunpeng.cn/blog_image/image-20220321212320107.png)

### 2.4：故障排查

有些时候，可能会出现ImagePullBackOff，原因就是镜像下载失败，需要手动下载

或者会出现其他问题，一般可以通过`kubectl describe pods pod_name -n monitoring`查看events解决



## 03：使用prometheus

默认的资源清单中service使用的是ClusterIP，我们在集群外部是访问不到的，这里可以有2种方法访问

- Ingress
  - 为prometheus-k8s、alertmanager-main、grafana创建Ingress
- NodePort
  - 对prometheus-service.yaml、grafana-service.yaml、alertmanager-service.yaml创建NodePort



### 3.1：配置NodePort

prometheus-service.yaml

```yaml
spec:
  type: NodePort
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 30200
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
```

grafana-service.yaml

```yaml
spec:
  type: NodePort
  ports:
  - name: http
    port: 3000
    targetPort: http
    nodePort: 30100
```

alertmanager-service.yaml

```yaml
spec:
  type: NodePort
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 30300
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
```

配置完成后，我们重新应用

```bash
kubectl apply -f prometheus-service.yaml -f grafana-service.yaml -f alertmanager-service.yaml
```

使用浏览器就能够访问了

### 3.2：配置Ingress

> 前提是k8s集群中已经部署好 nginx-ingress-controller

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prom-ingress
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: "nginx"
    prometheus.io/http_probe: "true"
spec:
  rules:
  - host: prom.linux98.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s
            port:
              number: 9090
```

应用资源清单，查看对应端口

![image-20220322001519519](https://image.lichunpeng.cn/blog_image/image-20220322001519519.png)

对访问主机做dns解析

```
192.168.31.11 prom.linux98.com
# 任意节点ip   绑定域名
```

使用浏览器访问http://192.168.31.11:Ingress_NodePort

![image-20220322001538951](https://image.lichunpeng.cn/blog_image/image-20220322001538951.png)

最后，有需要的对ingress进行负载均衡，这样就做到了prometheus在k8s中的高可用。



## 04：grafana配置

上面我们在外部已经可以访问prometheus了，使用相同方法为grafana也创建一个NodePort或Ingress上添加grafana

做好hosts解析

访问：grafana.linux98.com:Ingress_NodePort

默认用户名密码admin admin

默认已经为我们创建好了prometheus数据源，且使用的是集群内部的svc域名，做到了高可用

![image-20220322102348016](https://image.lichunpeng.cn/blog_image/image-20220322102348016.png)

点击之后，将自带的dashboard 引入

![image-20220322102445209](https://image.lichunpeng.cn/blog_image/image-20220322102445209.png)

对于更漂亮的ashboard我们可以使用 13105

![image-20220322102808877](https://image.lichunpeng.cn/blog_image/image-20220322102808877.png)

对于13105面板提示的修改配置

manifests/prometheusAdapter-serviceMonitor.yaml

```yaml
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    interval: 30s
    metricRelabelings:
    - action: drop
      regex: (apiserver_client_certificate_.*|apiserver_envelope_.*|apiserver_flowcontrol_.*|apiserver_storage_.*|apiserver_webhooks_.*|workqueue_.*)
      sourceLabels:
      - __name__
      # ------ 添加下面的部分
    - action: replace
      sourceLabels:
      - instance
      regex: (.+)
      separator: ;
      targetLabel: node
      replacement: $1  
```

应用资源清单，再使其重新加载配置文件

```bash
kubectl apply -f prometheusAdapter-serviceMonitor.yaml
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```



## 05：处理alert

0.10版本安装完成后，会出现很多警告，我们需要逐一修复警告。

![image-20220322185309769](https://image.lichunpeng.cn/blog_image/image-20220322185309769.png)

### 5.1：Watchdog

这个是Prometheus存活的一个状态传递信号，只要存活就一直发送告警，正常我们的监控环境都是互相监控的，这里就不需要这个告警机制了，我们用集群外部的zabbix或prometheus来监控k8s内部的promehteus。

/root/manifests/kubePrometheus-prometheusRule.yaml

找到下面的部分，注视掉

```yaml
    # - alert: Watchdog
    #   annotations:
    #     description: |
    #       This is an alert meant to ensure that the entire alerting pipeline is functional.
    #       This alert is always firing, therefore it should always be firing in Alertmanager
    #       and always fire against a receiver. There are integrations with various notification
    #       mechanisms that send a notification when this alert is not firing. For example the
    #       "DeadMansSnitch" integration in PagerDuty.
    #     runbook_url: https://runbooks.prometheus-operator.dev/runbooks/general/watchdog
    #     summary: An alert that should always be firing to certify that Alertmanager
    #       is working properly.
    #   expr: vector(1)
    #   labels:
    #     severity: none
```

重新应用资源清单，并使其重新加载配置文件

```bash
kubectl apply -f /root/manifests/kubePrometheus-prometheusRule.yaml
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```

### 5.2：CPUThrottlingHigh

默认为cpu的负载做了警告，但阈值为25%，我们修改为80%。

/root/manifests/kubernetesControlPlane-prometheusRule.yaml

```yaml
    - alert: CPUThrottlingHigh
      annotations:
        description: '{{ $value | humanizePercentage }} throttling of CPU in namespace
          {{ $labels.namespace }} for container {{ $labels.container }} in pod {{
          $labels.pod }}.'
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/cputhrottlinghigh
        summary: Processes experience elevated CPU throttling.
      expr: |
        sum(increase(container_cpu_cfs_throttled_periods_total{container!="", }[5m])) by (container, pod, namespace)
          /
        sum(increase(container_cpu_cfs_periods_total{}[5m])) by (container, pod, namespace)
          > ( 80 / 100 )
      for: 15m
      labels:
        severity: info
```

重新应用资源清单，并使其重新加载配置文件

```bash
kubectl apply -f /root/manifests/kubernetesControlPlane-prometheusRule.yaml
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```

### 5.3：controller-manager和scheduler

对于这两个组件的监控，详细信息在下面的文件中存放

```
/root/manifests/kubernetesControlPlane-serviceMonitorKubeControllerManager.yaml
/root/manifests/kubernetesControlPlane-serviceMonitorKubeScheduler.yaml
```

我们以scheduler为例

```yaml
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    interval: 30s
    port: https-metrics
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
  jobLabel: app.kubernetes.io/name
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app.kubernetes.io/name: kube-scheduler
```

从其中获取到有效信息

- 监控的协议为https
- 标签选择器筛选的为kube-system 这个namespace下的 app.kubernetes.io/name: kube-scheduler



所以我们查看这个service是否存在，我使用的是kubeadm安装，默认不存在，则创建

>  创建service

文件名称有意义自己能找到就好，我这里就临时使用01.yaml

```yaml
# ports 和 labels 要对应上面的监控资源清单内容
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
    app.kubernetes.io/name: kube-controller-manager
spec:
  selector:
    component: kube-controller-manager
  type: ClusterIP
  clusterIP: None
  ports:
  - name: https-metrics
    port: 10257
    targetPort: 10257
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    app.kubernetes.io/name: kube-scheduler
spec:
  selector:
    component: kube-scheduler
  type: ClusterIP
  clusterIP: None
  ports:
  - name: https-metrics
    port: 10259
    targetPort: 10259
    protocol: TCP
```

应用资源清单

```
kubectl apply -f 01.yaml
```

修改kube-controller-manager的监听地址为0.0.0.0，这里不用担心安全问题，k8s内部组件通信都是使用证书互相加密的。

```
sed -e "s/- --address=127.0.0.1/- --address=0.0.0.0/" -i /etc/kubernetes/manifests/kube-controller-manager.yaml
sed -e "s/- --address=127.0.0.1/- --address=0.0.0.0/" -i /etc/kubernetes/manifests/kube-scheduler.yaml
```

配置完成后，pod会自动重建，所有的master节点都要操作！

> 检查状态

```bash
# kubectl get svc -n kube-system
NAME                      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                        AGE
kube-controller-manager   ClusterIP   None         <none>        10257/TCP                      5h1m
kube-scheduler            ClusterIP   None         <none>        10259/TCP                      5h1m

# kubectl get ep -n kube-system
NAME                      ENDPOINTS                                                                  AGE
kube-controller-manager   192.168.31.12:10257,192.168.31.13:10257                                    5h3m
kube-scheduler            192.168.31.12:10259,192.168.31.13:10259                                    5h3m

# kubectl get ep kube-controller-manager -n kube-system -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  creationTimestamp: "2022-03-22T05:36:30Z"
  labels:
    app.kubernetes.io/name: kube-controller-manager
    service.kubernetes.io/headless: ""
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "59424"
  uid: 03188884-e980-4453-818a-fb8ed446a28a
subsets:
- addresses:
  - ip: 192.168.31.12
    nodeName: master02
    targetRef:
      kind: Pod
      name: kube-controller-manager-master02
      namespace: kube-system
      resourceVersion: "55697"
      uid: 9a4049b8-5e5d-4fa6-a77a-10c0d0538d47
  - ip: 192.168.31.13
    nodeName: master03
    targetRef:
      kind: Pod
      name: kube-controller-manager-master03
      namespace: kube-system
      resourceVersion: "53520"
      uid: a2423bdf-6293-46b4-a804-96ee41ea0c73
  ports:
  - name: https-metrics
    port: 10257
    protocol: TCP
# scheduler忽略
```

最后，检查prometheus的alerts

![image-20220322184258944](https://image.lichunpeng.cn/blog_image/image-20220322184258944.png)

### 5.4：KubeProxyDown

这个是0.9版本kube-prometheus加入的监控kube-proxy新功能。0.10发布的时候，对其禁用了，但没取消掉告警配置。

参考地址： https://github.com/prometheus-operator/kube-prometheus/pull/1609

我们直接注视掉rules即可，等待下个版本kube-proxy更加完善后再使用。

/root/manifests/kubernetesControlPlane-prometheusRule.yaml

```yaml
  # - name: kubernetes-system-kube-proxy
  #   rules:
  #   - alert: KubeProxyDown
  #     annotations:
  #       description: KubeProxy has disappeared from Prometheus target discovery.
  #       runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubeproxydown
  #       summary: Target disappeared from Prometheus target discovery.
  #     expr: |
  #       absent(up{job="kube-proxy"} == 1)
  #     for: 15m
  #     labels:
  #       severity: critical
```

重新应用资源清单，并删除prometheus-k8s的pod，使其重新加载配置文件

```bash
kubectl apply -f /root/manifests/kubernetesControlPlane-prometheusRule.yaml
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```

## 06：配置告警

alertmanager的规则在这里配置: /root/manifests/alertmanager-secret.yaml

示例文件

```yaml
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "5m"
      "smtp_smarthost": "smtp.126.com:25"
      "smtp_from": "linux98_mail@126.com"
      "smtp_auth_username": "linux98_mail@126.com"
      "smtp_auth_password": "xxxxxxxxxxxxxxx"
      "smtp_hello": "126.com"
      "smtp_require_tls": false
    "inhibit_rules":
    - "equal":
      - "namespace"
      - "alertname"
      "source_matchers":
      - "severity = critical"
      "target_matchers":
      - "severity =~ warning|info"
    - "equal":
      - "namespace"
      - "alertname"
      "source_matchers":
      - "severity = warning"
      "target_matchers":
      - "severity = info"
    "receivers":
    - "name": "Default"
    - "name": "Watchdog"
    - "name": "Critical"
    - "name": "email"
      "email_configs":
      - "to": "linux98_mail@yeah.net"
        "send_resolved": true
    "route":
      "group_by":
      - "namespace"
      - "alertname"
      "group_interval": "30s"
      "group_wait": "30s"
      "receiver": "Default"
      "repeat_interval": "60s"
      "routes":
      - "matchers":
        - "alertname = Watchdog"
        "receiver": "Watchdog"
      - "matchers":
        - "severity = critical"
        "receiver": "email"
```

prometheus的rules文件有很多，搜索文件名关于rule的就是了，修改对应的即可

这里使用kube-controller-manager 和scheduler作为演示

/root/manifests/kubernetesControlPlane-prometheusRule.yaml

```yaml
  - name: kubernetes-system-scheduler
    rules:
    - alert: KubeSchedulerDown
      annotations:
        description: KubeScheduler has disappeared from Prometheus target discovery.
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubeschedulerdown
        summary: Target disappeared from Prometheus target discovery.
      expr: |
        absent(up{job="kube-scheduler"} == 1)
      for: 15s
      labels:
        severity: critical
  - name: kubernetes-system-controller-manager
    rules:
    - alert: KubeControllerManagerDown
      annotations:
        description: KubeControllerManager has disappeared from Prometheus target
          discovery.
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubecontrollermanagerdown
        summary: Target disappeared from Prometheus target discovery.
      expr: |
        absent(up{job="kube-controller-manager"} == 1)
      for: 15s
      labels:
        severity: critical
```

调整pending时间为15s，获取到错误马上就告警了

```bash
kubectl apply -f /root/manifests/kubernetesControlPlane-prometheusRule.yaml
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```

最后告警效果

![image-20220322204656348](https://image.lichunpeng.cn/blog_image/image-20220322204656348.png)

## 07：配置自定义监控

对于这个版本的kube-prometheus来说，创建serviceMonitor资源时，必须有2个专属标签，才可以被kube-prometheus监控到，并自动重载配置文件。

### 7.1：为kube-prometheus开放权限

mykube/kube-prometheus/manifests/prometheus-clusterRole.yaml

原文件

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.32.1
  name: prometheus-k8s
  namespace: monitoring
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
```

修改后

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.32.1
  name: prometheus-k8s
  namespace: monitoring
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  - services   # 增加
  - endpoints  # 增加
  - pods       # 增加
  verbs:
  - get
  - list       # 增加
  - watch      # 增加
- nonResourceURLs:
  - /metrics
  verbs:
  - get

```

应用后，重新建立新的prometheus-pod

```bash
kubectl delete pods -n monitoring prometheus-k8s-0
kubectl delete pods -n monitoring prometheus-k8s-1
```

### 7.2：创建演示deployment

mykube/kube-prometheus/myapp-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-monitoring
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myweb-monitor
      release: v0.1
  template:
    metadata:
      labels:
        app: myweb-monitor
        release: v0.1
    spec:
      containers:
      - name: myweb
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "100m"

```

### 7.3：创建演示service

mykube/kube-prometheus/myapp-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb-monitor
  namespace: default
  labels:
    app: myweb-monitor
spec:
  selector:
    app: myweb-monitor
    release: v0.1
  ports:
  - name: http
    port: 80
    targetPort: 80

```



### 7.4：创建演示serviceMonitor

> 这里创建完成后，kube-prometheus会自动重载配置文件

mykube/kube-prometheus/myapp-servicemonitor.yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-prometheus-servicemonitor
  namespace: monitoring
  labels:
    # 这两个标签很重要，prometheus会监测这两个标签，来进行读取和重载prometheus配置
    app.kubernetes.io/name: myapp-prometheus-servicemonitor  # service名称
    app.kubernetes.io/part-of: kube-prometheus
spec:
  selector:
    matchLabels:  # 使用标签选择器查找需要监控的service
      app: myweb-monitor
  jobLabel: app.kubernetes.io/name  # job标签由哪个值显示
  namespaceSelector:  # service在哪个namespace中
    matchNames:
    - default
  endpoints:  # service的端口，这里可以使用service的端口别名
  - port: http
    scheme: http  # 用于抓取的http协议，http或https

```

### 7.5：配置监控规则

在这个文件中的 

mykube/kube-prometheus/manifests/prometheus-prometheusRule.yaml

这个部分下面添加自己的规则

```yaml
spec:
  groups:
  - name: prometheus
    # 增加规则，示例文件可以看原有的规则
```

重载prometheus配置文件

```bash
# 重载配置文件
curl -X POST http://prom.linux98.com:30080/-/reload
```



### 7.6：刷新web查看

由于我这里没有在pod中配置metrics，所以无法获取监控指标，但是target已经可以监控了

![image-20220402213225745](https://image.lichunpeng.cn/blog_image/image-20220402213225745.png)