---
description: nexus
---

# 🇧🇾 nexus

## 1、基本介绍

### 1.1、nexus是什么？

Nexus 是一个专门的 Maven 仓库管理软件，它不仅能搭建 Maven 私服，还具备如下一些优点使其日趋成为最流行的 Maven 仓库管理器：

* 提供了强大的仓库管理功能，构件搜索功能
* 它基于 REST，友好的 UI 是一个 ext.js 的 REST 客户端
* 它占用较少的内存
* 基于简单文件系统而非数据库

### 1.2 如果没有私服会出现的问题

* 如果没有私服，我们所需的所有构件都需要通过 Maven 的中央仓库或者第三方的 Maven 仓库下载到本地，而一个团队中的所有人都重复的从 Maven 仓库下载构件无疑加大了仓库的负载和浪费了外网带宽，如果网速慢的话，还会影响项目的进程。
* 另外，很多情况下项目的开发都是在内网进行的，可能根本连接不了 Maven 的中央仓库和第三方的 Maven 仓库。
* 我们开发的公共构件如果需要提供给其它项目使用，也需要搭建私服。
* npm、go、docker镜像都是同样的问题

### 1.3 搭建私服的优点

&#x20;Maven 私服的概念就是在本地架设一个 Maven 仓库服务器，在代理远程仓库的同时维护本地仓库。当我们需要下载一些构件（artifact）时，如果本地仓库没有，再去私服下载，私服没有，再去中央仓库下载。这样做会有如下一些优点：

* 减少网络带宽流量
* 加速 Maven 构建
* 部署第三方构件
* 提高稳定性、增强控制
* 降低中央仓库的负载

## 2、nexus 安装部署

### 2.1 简单安装（docker-compose安装）

{% code title="docker-compose.yaml" %}
```yaml
version: '3'
services:
  nexus3:
    image: 'sonatype/nexus3'
    container_name: nexus3
    restart: always
    volumes:
      - ./nexus-data:/nexus-data
    ports:
      - '8081:8081'
      - '5000:5000'
      - '5001:5001'
      - '8181:8181'
```
{% endcode %}

启动 docker-compose up -d

关闭 docker-compose down

### 2.2 k8s 安装

{% tabs %}
{% tab title="pv-pvc.yaml" %}
```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nexus-pv
spec:
  capacity:
    storage: 500Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: local-storage-nexus
  local:
    path: /data/nexus_data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-node04
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nexus-pvc
  namespace: middleware-test
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Gi
  storageClassName: local-storage-nexus
  volumeMode: Filesystem
  volumeName: nexus-pv
```
{% endtab %}

{% tab title="deployment.yaml" %}
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: nexus3
  name: nexus3
  namespace: middleware-test
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: nexus3
  template:
    metadata:
      labels:
        k8s-app: nexus3
      name: nexus3
      namespace: middleware-test
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node04 #指定调度节点为带有label标记为：cloudnil.com/role=dev的node节点
      containers:
      - name: nexus3
        image: harbor.idx.space/ops/nexus3:latest
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 8081
            name: web
            protocol: TCP
        # livenessProbe:
        #   httpGet:
        #     path: /
        #     port: 8081
        #   initialDelaySeconds: 540
        #   periodSeconds: 30
        #   failureThreshold: 6
        # readinessProbe:
        #   httpGet:
        #     path: /
        #     port: 8081
        #   initialDelaySeconds: 540
        #   periodSeconds: 30
        #   failureThreshold: 6
        resources:
          limits:
            cpu: 4000m
            memory: 4Gi
          requests:
            cpu: 500m
            memory: 512Mi
        volumeMounts:
        - name: nexus-data
          mountPath: /nexus-data/
          subPath: nexus-data
        - name: nexus-data
          mountPath: /sonatype-work/
          subPath: data
      volumes:
        - name: nexus-data
          persistentVolumeClaim:
            claimName: nexus-pvc

```
{% endtab %}

{% tab title="svc.yaml" %}
```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nexus3
  name: nexus3
  namespace: middleware-test
spec:
  ports:
    - name: http
      port: 8081        #服务端口
      protocol: TCP
      targetPort: 8081
  selector:
    k8s-app: nexus3
  type: NodePort

```
{% endtab %}

{% tab title="ingress.yaml" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"
  name: nexus3
  namespace: middleware-test
spec:
  ingressClassName: nginx
  rules:
  - host: nexus.idx.space
    http:
      paths:
      - backend:
          service:
            name: nexus3
            port:
              number: 8081
        pathType: ImplementationSpecific
```
{% endtab %}
{% endtabs %}

## 3、配置

### 3.1 yum仓库

设置proxy

centos: [https://mirrors.aliyun.com/centos/](https://mirrors.aliyun.com/centos/)

epel: [https://mirrors.aliyun.com/epel](https://mirrors.aliyun.com/epel)

kubernetes: [https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86\_64/](https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86\_64/)

docker-ce: [https://download.docker.com/linux/centos/](https://download.docker.com/linux/centos/)

```
[os]
name=os
baseurl=https://nexus.idx.space/repository/centos/\$releasever/os/\$basearch/
enabled=1
gpgcheck=0
[updates]
name=updates
baseurl=https://nexus.idx.space/repository/centos/\$releasever/updates/\$basearch/
enabled=1
gpgcheck=0
[extras]
name=extras
baseurl=https://nexus.idx.space/repository/centos/\$releasever/extras/\$basearch/
enabled=1
gpgcheck=0
[centosplus]
name=centosplus
baseurl=https://nexus.idx.space/repository/centos/\$releasever/centosplus/\$basearch/
enabled=1
gpgcheck=0
[configmanagement]
name=configmanagement
baseurl=https://nexus.idx.space/repository/centos/\$releasever/configmanagement/\$basearch/ansible-29/
enabled=1
gpgcheck=0
[epel]
name=Extra Packages for Enterprise Linux 7 - \$basearch
baseurl=https://nexus.idx.space/repository/epel/7/\$basearch
enabled=1
gpgcheck=0
[docker-ce]
name=docker-ce
baseurl=https://nexus.idx.space/repository/docker-ce/$releasever/$basearch/stable/
enabled=1
gpgcheck=0
[kubernetes]
name=kubernetes
baseurl=https://nexus.idx.space/repository/kubernetes/
enabled=1
gpgcheck=0
```





