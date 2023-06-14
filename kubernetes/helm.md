# 🫐 Helm

## 1、helm3

移除 Tiller（Helm 2 是一种 Client-Server 结构，客户端称为 Helm，服务器称为 Tiller）。Helm 3 只有客户端结构，客户端仍称为 Helm。如下图所示，它的操作类似于 Helm 2 客户端，但客户端直接与 Kubernetes API 服务器交互。

### 1.1、helm3架构

![helm3架构图](../.gitbook/assets/helm.jpg)

### **1.2、新特性**

* 支持 Helm 图表新版本
* 支持库图表
* Release 以新格式存储
* 支持在 OCI 注册表中存储 Helm 图表(实验性)
* 现在可以根据 JSON 模式验证图表提供的值
* 支持 XDG 基目录规范
* 不需要初始化 Helm
* 改进版本升级策略
* 简化 CRD 支持
* Helm 测试框架更新
* 仍支持 Helm 2 接口

### **1.3、更改存储库**

Helm 3 改进了存储库的体验。在 Helm 2 中，默认情况下包含图表存储库。在 Helm 3 中，默认情况下不包含任何存储库。因此，你首先需要做的事情之一就是添加一个存储库。

### 1.4、下载安装

```
wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz
tar -zxvf helm-v3.2.4-linux-amd64.tar.gz
cp linux-amd64/helm  /usr/local/bin/helm

[root@k8s-master helm]# helm version
version.BuildInfo{Version:"v3.2.4", GitCommit:"0ad800ef43d3b826f31a5ad8dfbb4fe05d143688", GitTreeState:"clean", GoVersion:"go1.13.12"}
```

### 1.5、常用helm命令

```
# 安装时候debug
helm init debug
# 卸载helm
helm reset -f
# 移除仓库
helm repo remove stable
# 新增仓库
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
# 更新仓库
helm repo update
# 列出仓库
helm repo list
# 查询仓库
helm search
```

## 2、helm2（过时）

官网[https://helm.sh/](https://helm.sh/)

### 2.1、下载安装包

```bash
cd /data/kubernetes/heml/
wget https://get.helm.sh/helm-v2.16.9-linux-amd64.tar.gz
tar -zxvf helm-v2.16.9-linux-amd64.tar.gz
cp linux-amd64/helm  /usr/local/bin/
```

#### 配置serviceaccount

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: helm-serviceaccount
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: helm-serviceaccount
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: helm-serviceaccount
    namespace: kube-system
```

`kubectl apply -f helm-serviceaccount.yaml`

#### 初始化

```bash
helm init
```

#### 查看版本

```
helm version
[root@k8s-master helm]# helm version
Client: &version.Version{SemVer:"v2.16.9", GitCommit:"8ad7037828e5a0fca1009dabe290130da6368e39", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.9", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```



#### 卸载

```
kubectl get -n kube-system secrets,sa,clusterrolebinding -o name|grep tiller|xargs kubectl -n kube-system delete
kubectl get all -n kube-system -l app=helm -o name|xargs kubectl delete -n kube-system
```
