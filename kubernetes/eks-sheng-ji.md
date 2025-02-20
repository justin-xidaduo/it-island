# 🥭 eks升级

AWS 会对每个 EKS 版本提供14个月支持，同时 AWS 会强制更新太旧的 EKS 集群，因此可能导致生产服务中断。要升级 EKS 集群，首先要注意以下事项：

* 验证与应用程序关联的容器映像版本是否可用，主要是排除应用异常导致的干扰。
* 检查集群所在子网有足够的可用 IP，可用IP不足也可能会导致 EKS 升级失败。
* 检查 node group 配置中的最大不可用性节点数量，这个数量表示每次更新期间节点组中可以替换的节点数。在大规模集群中，设置为百分比更有利于提高更新的速度。

## 一、升级EKS的检查工作

EKS 不允许跨版本升级，在升级 EKS 集群之前需要做兼容性的检查：

### 1.1 与 Kubernetes 相关的变更

每一个版本的 EKS 都有一些 API 已被弃用。 因此需要检查特定版本更改的详细信息，需要参考 Kubernetes 的变更日志和 EKS 版本更新文档。

* [https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG) 需要重点关注 Urgent Upgrade Notes 部分
* [https://docs.amazonaws.cn/eks/latest/userguide/kubernetes-versions.html](https://docs.amazonaws.cn/eks/latest/userguide/kubernetes-versions.html)

### 1.2 使用 kubent 检查 API 变更

国际版的 EKS web console 中提供了 Upgrade insights 工具，可以检查 API 变动。但是中国区没有这个工具，可以使用第三方工具 kube-no-trouble 进行检查。

1. 下载 [https://github.com/doitintl/kube-no-trouble/releases](https://github.com/doitintl/kube-no-trouble/releases) 并然后解压。
2.  确保 kubectl 当前的 context 是要升级的 EKS 集群，使用下面的cmd来检查：

    ```bash
    kubectl config current-context
    ```
3. 执行./kubent开始检查。 确保 >>> Deprecated APIs removed in 1.XX <<< 中没有任何内容。\
   注意：要保证 aws cli 的版本应大于2.0，尽量使用新版本的 aws cli 。

如果API版本发生变化，需要参考 [https://docs.amazonaws.cn/en\_us/eks/latest/userguide/update-cluster.html](https://docs.amazonaws.cn/en_us/eks/latest/userguide/update-cluster.html) 进行更新。

### 1.3 检查 addons 版本是否需要更新

所有的 addon 都在 kube-system namesapce中，首先需要确定所有 addon 需要升级的目标版本号，供后续更新使用。

#### 1.3.1 检查兼容的版本

使用以下命令可以列出与 EKS 1.22 版本兼容的所有插件版本：

1.  查看 kube-proxy 全部兼容的版本

    ```bash
    aws eks describe-addon-versions --kubernetes-version 1.22 --addon-name kube-proxy | grep addonVersion
    ```
2.  查看 coredns 兼容的版本

    ```bash
    aws eks describe-addon-versions --kubernetes-version 1.22 --addon-name coredns | grep addonVersion
    ```
3.  查看 vpc-cni 兼容的版本

    ```bash
    aws eks describe-addon-versions --kubernetes-version 1.22 --addon-name vpc-cni | grep addonVersion
    ```

    另外，建议结合 AWS 的文档确定最终要的 addon 版本：

[https://docs.aws.amazon.com/eks/latest/userguide/managing-kube-proxy.html](https://docs.aws.amazon.com/eks/latest/userguide/managing-kube-proxy.html)\
[https://docs.aws.amazon.com/eks/latest/userguide/managing-coredns.html](https://docs.aws.amazon.com/eks/latest/userguide/managing-coredns.html)\
[https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html)

#### 1.3.2 检查当前 addons 版本

您可以使用下面的 AWS CLI 命令来查询 addon的 image 版本：

```bash
#kube-proxy
kubectl get daemonset kube-proxy -n kube-system -o yaml | grep image

# coredns
kubectl get deployment coredns -n kube-system -o yaml | grep image

# vpc-cni
kubectl get daemonset aws-node -n kube-system -o yaml | grep image
```

对比一下当前的 addon 版本是否则在兼容列表中，如果不在就需要升级 addon 版本。如果使用了其他插件，也需要查看相关文档以确定兼容性。

## 二、更新 EKS 版本

EKS 不允许跨版本升级，所以在升级过程中，需要先升级到1.22版本，然后升级到1.23，最后升级到1.24，以此类推。\
按照 EKS Control Plane -> EKS addons -> EKS node group 的顺序进行升级。

### 2.1 升级 EKS Control Plane

升级的第一步是先升级 EKS Control Plane，这一步很简单，只需要在 eks web console 中点击 “update now” 更新即可。一旦升级成功就无法降级到以前的版本，单独升级 EKS Control Plane 不会重新调度集群中的 Pod，只有升级 node group 时 Pod 才会被重新调度。

### 2.2 更新插件到兼容的版本

通常需要更新包括 kube-proxy、coredns、vpc-cni 等 addon 。如果 addon 是通过 eks web console 安装的，也可以在 web console 中直接进行升级。

如果插件是 self-managed 的，那么必须找到正确的版本，然后使用 `kubectl set image` 进行更新。

#### 2.2.1 更新 kube-proxy

以中国区为例，首先打开 [https://docs.amazonaws.cn/eks/latest/userguide/managing-kube-proxy.html](https://docs.amazonaws.cn/eks/latest/userguide/managing-kube-proxy.html) ， 需要在文档中找到 kube-proxy 在 ecr 中的镜像地址和 tag 。例如中国北京区的地址是：918309763551.dkr.ecr.cn-north-1.amazonaws.com.cn/eks/kube-proxy tag 是： v1.25.16-minimal-eksbuild.1

查看当前使用的 kube-proxy image ：

```bash
kubectl get daemonset kube-proxy --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}'
```

更新 image ：

```bash
kubectl set image daemonset.apps/kube-proxy -n kube-system kube-proxy=918309763551.dkr.ecr.cn-north-1.amazonaws.com.cn/eks/kube-proxy:v1.25.16-minimal-eksbuild.1
```

命令执行后 kube-proxy 的 pod 就会使用新 image 重建，需要检查 pods 是否成功重建。

```bash
kubectl get pods -n kube-system
```

#### 2.2.2 更新 coredns

同 kube-proxy 类似，访问 [https://docs.amazonaws.cn/eks/latest/userguide/managing-coredns.html](https://docs.amazonaws.cn/eks/latest/userguide/managing-coredns.html) 找到 ecr 中coredns 的地址和 tag 。

查看当前的 image ：

```bash
kubectl get deployment coredns --namespace kube-system -o=jsonpath='{$.spec.template.spec.containers[:1].image}'
```

更新 coredns image：

```bash
kubectl set image --namespace kube-system deployment.apps/coredns coredns=918309763551.dkr.ecr.region-code.amazonaws.com.cn/eks/coredns:v1.8.7-eksbuild.3
```

#### 2.2.3 更新 vpc-cni

vpc-cni的更新不能使用上面的方式，首先访问 [https://github.com/aws/amazon-vpc-cni-k8s/releases](https://github.com/aws/amazon-vpc-cni-k8s/releases) ，找到要升级的版本，下载对应的 yaml 文件，以v1.16.3 为例：\
aws 中国区：[https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/v1.16.3/config/master/aws-k8s-cni-cn.yaml](https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/v1.16.3/config/master/aws-k8s-cni-cn.yaml)\
aws 国际区：[https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/v1.16.3/config/master/aws-k8s-cni.yaml](https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/v1.16.3/config/master/aws-k8s-cni.yaml)

需要确认 yaml 文件中： image 链接的 account numbmer/region code/release tag是否正确，如果下载了对应区域的 yaml 文件通常不需要修改。如果需要修改为防止出错，建议使用 sed 命令全部一次性修改：

```bash
sed -i.bak -e 's|cn-northwest-1|cn-north-1|' aws-k8s-cni-cn.yaml
```

更新 vpc-cni 插件：

```bash
kubectl apply -f aws-k8s-cni-cn.yaml
```

### 2.3 更新节点组

通常使用的是 aws managed 的 node group ，在升级完 Control Plane 后，在 node group 设置页直接点击 “Update now” 即可。可以选择 rolling update 或者 force update ，rolling update 会先调度 pods 到新节点，因此不会中断业务，但是会更耗时。对 self-managed node group 需要创建新节点组，将应用程序/Pod 迁移到新节点组，然后删除旧节点组。

## 3. EKS 升级后的检查

需要检查：

1. 验证Control Plane 和节点组版本是否已更新。
2. 检查插件的状态。
3. 检查节点状态和版本。
4. 检查应用是否正常。

至此 AWS EKS 的版本升级就结束了。总体来说对于简单的环境不是特别复杂，升级的风险总体可控，风险不是不大。
