# kubeadm安装   kubernetes-1.18.2

### 开启ipvs

```
# kube-proxy开启ipvs的前置条件

cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs 
modprobe -- ip_vs_rr 
modprobe -- ip_vs_wrr 
modprobe -- ip_vs_sh 
modprobe -- nf_conntrack_ipv4 
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

### 安装docker

```
# 安装docker
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
yum update -y && yum install -y docker-ce

# 创建/etc/docker目录
mkdir /etc/docker
# 配置daemon.josn
cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {"max-size": "100m"},
    "registry-mirrors": ["https://odhacnra.mirror.aliyuncs.com"]
} 
EOF

mkdir -p /etc/systemd/system/docker.service.d 
# 重启docker服务 
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

### 安装kubeadm

```
# 安装kubeadm

cat << EOF > /etc/yum.repos.d/kubernetes.repo 
[kubernetes] 
name=Kubernetes 
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1 
gpgcheck=0 
repo_gpgcheck=0 
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
EOF
yum -y install  kubeadm-1.18.2 kubectl-1.18.2 kubelet-1.18.2
systemctl enable kubelet.service
```

### 配置kubeadm自动补全

```
#安装bash自动补全插件
yum install bash-completion -y
#设置kubectl与kubeadm命令补全，下次login生效
kubectl completion bash >/etc/bash_completion.d/kubectl
kubeadm completion bash > /etc/bash_completion.d/kubeadm
```

### 控制节点

获取kubeadm默认配置文件,配置文件如kubeadm-config.yaml



修改配置文件并执行以下命令,执行结果如kubeadm-init.log

`kubeadm init --config=kubeadm-config.yaml |tee kubeadm-init.log`

{% tabs %}
{% tab title="kubeadm-config.yaml" %}
```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.0.0.31
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.18.2
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16
scheduler: {}

# 默认调度算法改为ipvs
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
```
{% endtab %}

{% tab title="kubeadm-init.log" %}
```
[init] Using Kubernetes version: v1.18.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.31]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [10.0.0.31 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [10.0.0.31 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 20.006653 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.31:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:9ce259686edf2229293bc76060a8fdf48be381246283233010acb841e66ebb38
```
{% endtab %}
{% endtabs %}

#### 配置kubeadm命令可使用

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config极端
```

#### 安装flannel，yaml如下：

{% embed url="https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml" %}
kube-flannel.yml
{% endembed %}

#### [kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)

```
 kubectl apply -f  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 计算节点

#### 增加另外两个node节点，命令见`kubeadm-init.log`

```
kubeadm join 10.0.0.31:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:9ce259686edf2229293bc76060a8fdf48be381246283233010acb841e66ebb38
```

### 查询状态

```
[root@k8s-master ~]# kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   33m   v1.18.2
k8s-node1    Ready    <none>   15m   v1.18.2
k8s-node2    Ready    <none>   15m   v1.18.2

[root@k8s-master ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-8flkg             1/1     Running   0          32m
kube-system   coredns-7ff77c879f-vww6d             1/1     Running   0          32m
kube-system   etcd-k8s-master                      1/1     Running   0          33m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          33m
kube-system   kube-controller-manager-k8s-master   1/1     Running   1          33m
kube-system   kube-flannel-ds-amd64-krwhk          1/1     Running   0          2m1s
kube-system   kube-flannel-ds-amd64-r87pl          1/1     Running   0          2m1s
kube-system   kube-flannel-ds-amd64-z47hs          1/1     Running   0          2m3s
kube-system   kube-proxy-dpl5g                     1/1     Running   0          15m
kube-system   kube-proxy-hdldk                     1/1     Running   0          32m
kube-system   kube-proxy-sn9w7                     1/1     Running   0          15m
kube-system   kube-scheduler-k8s-master            1/1     Running   1          33m

[root@k8s-master ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:48:36Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

至此已完成安装

### 增加新的node

```bash
kubeadm token create --print-join-command
# 根据输出增加新的node即可
# kubeadm join 172.16.10.7:6443 --token 6r5tir.x8d0mtki6slle6vl     --discovery-token-ca-cert-hash sha256:eed7dbd8c13dc8935ebf0d79e62db371d7cf1b6f402adbda5ba67e6e4747c213
```

### 增加新的master

```bash
# 在master上生成用于新master加入的证书
kubeadm init phase upload-certs --experimental-upload-certs

#添加新master，把生成ID加到--experimental-control-plane --certificate-key后。
kubeadm join 172.31.182.156:8443  --token ortvag.ra0654faci8y8903 \
  --discovery-token-ca-cert-hash sha256:04755ff1aa88e7db283c85589bee31fabb7d32186612778e53a536a297fc9010 \
  --experimental-control-plane --certificate-key f8d1c027c01baef6985ddf24266641b7c64f9fd922b15a32fce40b6b4b21e47d
```
