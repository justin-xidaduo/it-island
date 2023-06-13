# install helm-2.16

官网[https://helm.sh/](https://helm.sh/)

#### 下载安装包

```bash
cd /data/kubernetes/heml/
wget https://get.helm.sh/helm-v2.16.9-linux-amd64.tar.gz
tar -zxvf helm-v2.16.9-linux-amd64.tar.gz
cp linux-amd64/helm  /usr/local/bin/
```

配置serviceaccount

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

#### 其它操作

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

#### 卸载

```
kubectl get -n kube-system secrets,sa,clusterrolebinding -o name|grep tiller|xargs kubectl -n kube-system delete
kubectl get all -n kube-system -l app=helm -o name|xargs kubectl delete -n kube-system
```
