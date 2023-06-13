---
description: k8s 健康检查
---

# 🍏 Pod HealthCheck

## 1 介绍说明

应用在运行过程中难免会出现错误，如程序异常，软件异常，硬件故障，网络故障等，kubernetes提供Health Check健康检查机制，当发现应用异常时会自动重启容器，将应用从service服务中剔除，保障应用的高可用性。k8s 定义了3种健康检查机制

* readiness probes 准备就绪检查，通过readiness是否准备接受流量，准备完毕加入到endpoint，否则剔除
* liveness probes 在线检查机制，检查应用是否可用，如死锁，无法响应，异常时会自动重启容器
* tartup probes 启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被前面的探针kill掉

每种探测机制支持三种健康检查方法，分别是命令行exec，httpGet和tcpSocket，其中exec通用性最强，适用与大部分场景，tcpSocket适用于TCP业务，httpGet适用于web业务。

* exec 提供命令或shell的检测，在容器中执行命令检查，返回码为0健康，非0异常
* httpGet http协议探测，在容器中发送http请求，根据http返回码判断业务健康情况
* tcpSocket tcp协议探测，向容器发送tcp建立连接，能建立则说明正常

每种探测方法能支持几个相同的检查参数，用于设置控制检查时间：

* initialDelaySeconds 初始第一次探测间隔，用于应用启动的时间，防止应用还没启动而健康检查失败
* periodSeconds 检查间隔，多久执行probe检查，默认为10s
* timeoutSeconds 检查超时时长，探测应用timeout后为失败
* successThreshold 成功探测阈值，表示探测多少次为健康正常，默认探测1次

## 2 示例

### 2.1 liveness 存活检测

```yaml

# exec 方式
---
apiVersion: v1
kind: Pod
metadata:
  name: exec-liveness-probe
  annotations:
    kubernetes.io/description: "exec-liveness-probe"
spec:
  containers:
  - name: exec-liveness-probe
    image: centos:latest
    imagePullPolicy: IfNotPresent
    args:    #容器启动命令，生命周期为30s
    - /bin/sh
    - -c
    - touch /tmp/liveness-probe.log && sleep 10 && rm -f /tmp/liveness-probe.log && sleep 20
    livenessProbe:
      exec:  #健康检查机制，通过ls -l /tmp/liveness-probe.log返回码判断容器的健康状态
        command:
        - ls
        - l
        - /tmp/liveness-probe.log
      initialDelaySeconds: 1
      periodSeconds: 5
      timeoutSeconds: 1


# http 健康检查
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-httpget-livess-readiness-probe
  annotations:
    kubernetes.io/description: "nginx-httpGet-livess-readiness-probe"
spec:
  containers:
  - name: nginx-httpget-livess-readiness-probe
    image: nginx:latest
    ports:
    - name: http-80-port
      protocol: TCP
      containerPort: 80
    livenessProbe:   #健康检查机制，通过httpGet实现实现检查
      httpGet:
        port: 80
        scheme: HTTP
        path: /index.html
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3


# tcpSocket
apiVersion: v1
kind: Pod
metadata:
  name: nginx-tcp-liveness-probe
  annotations:
    kubernetes.io/description: "nginx-tcp-liveness-probe"
spec:
  containers:
  - name: nginx-tcp-liveness-probe
    image: nginx:latest
    ports:
    - name: http-80-port
      protocol: TCP
      containerPort: 80
    livenessProbe:  #健康检查为tcpSocket，探测TCP 80端口
      tcpSocket:
       port: 80
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3


```

### 2.2 readness 就绪检测

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-tcp-liveness-probe
  annotations:
    kubernetes.io/description: "nginx-tcp-liveness-probe"
  labels:  #需要定义labels，后面定义的service需要调用
    app: nginx
spec:
  containers:
  - name: nginx-tcp-liveness-probe
    image: nginx:latest
    ports:
    - name: http-80-port
      protocol: TCP
      containerPort: 80
    livenessProbe:  #存活检查探针
      httpGet:
        port: 80
        path: /index.html
        scheme: HTTP
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3
    readinessProbe:  #就绪检查探针
      httpGet:
        port: 80
        path: /test.html
        scheme: HTTP
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3

```

### 2.3 startprob

应用程序将会有最多 5 分钟(30 \* 10 = 300s) 的时间来完成它的启动。 一旦启动探测成功一次，存活探测任务就会接管对容器的探测，对容器死锁可以快速响应。 如果启动探测一直没有成功，容器会在 300 秒后被杀死，并且根据 restartPolicy 来设置 Pod 状态

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-tcp-liveness-probe
  annotations:
    kubernetes.io/description: "nginx-tcp-liveness-probe"
  labels:  #需要定义labels，后面定义的service需要调用
    app: nginx
spec:
  containers:
  - name: nginx-tcp-liveness-probe
    image: nginx:latest
    ports:
    - name: http-80-port
      protocol: TCP
      containerPort: 80
    livenessProbe:  #存活检查探针
      httpGet:
        port: 80
        path: /index.html
        scheme: HTTP
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3
    readinessProbe:  #就绪检查探针
      httpGet:
        port: 80
        path: /test.html
        scheme: HTTP
      initialDelaySeconds: 3
      periodSeconds: 10
      timeoutSeconds: 3
    startupProbe:
      httpGet:
        path: /healthz
        port: liveness-port
        failureThreshold: 30
        periodSeconds: 10

```
