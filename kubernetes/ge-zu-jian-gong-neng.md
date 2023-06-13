# 🍐 各组件功能

### 1、kube-apiserver <a href="#item-0-4" id="item-0-4"></a>

它负责Kubernetes"资源组/资源版本/资源"以RESTful风格的形式对外暴露并提供服务。kube-apiserver组件也是集群中唯一与Etcd集群进行交互的核心组件。Kubernetes将所有的数据存储在Etcd集群中前缀为registry的目录下。\
kube-apiserver的特性：

* 将Kubernetes系统中所有资源对象都封装成RESTful风格的API接口进行管理。
* 可进行集群状态管理和数据管理，是唯一与Etcd集群交互的组件。
* 拥有丰富的集群安全访问机制，以及认证，授权及准入控制器。
* 提供了集群各组件的通信和交互功能。\


### 2、kube-scheduler

Kubernetes调度器是一个控制面进程，负责将Pods指派到节点上

通过调度算法为待调度Pod列表的每个Pod从可用Node列表中选择一个最适合的Node，并将信息写入etcd中

node节点上的kubelet通过API Server监听到kubernetes Scheduler产生的Pod绑定信息，然后获取对应的Pod清单，下载Image，并启动容器。

策略：

* LeastRequestedPriority 优先从备选节点列表中选择资源消耗最小的节点（CPU+内存）
* CalculateNodeLabelPriority 优先选择含有指定Label的节点
* BalancedResourceAllocation 优先从备选节点列表中选择各项资源使用率最均衡的节点



### 3、kube-controller-manager

Controller Manager还包括一些子控制器（副本控制器、节点控制器、命名空间控制器和服务账号控制器等），控制器作为集群内部的管理控制中心，负责集群内的Node、Pod副本、服务端点（Endpoint）、命名空间（Namespace）、服务账号（ServiceAccount）、资源定额（ResourceQuota）的管理，当某个Node意外宕机时，Controller Manager会及时发现并执行自动化修复流程，确保集群中的pod副本始终处于预期的工作状态

* controller-manager控制器每间隔5秒检查一次节点的状态
* 如果controller-manager控制器没有收到自节点的心跳，则将该node节点标记为不可达
* controller-manager将在标记为无法访问之前等待40秒
* 如果该node节点被标记为无法访问后5分钟还没有恢复，controller-manager会删除当前node节点的所有pod并在其它可用节点重建这些pod

pod高可用机制：

* node monitor period：节点监事周期 5s
* node monitor grace period:节点监视器宽限期 40s
* &#x20;pod eviction timeout: pod驱逐超时时间 5m

### 4、kube-proxy

Kubernetes 网络代理运行在node上，它反映了node上Kubernetes API中定义的服务，并可以通过一组后端进行简单的TCP、UDP和SCTP流转发或者在一组后端进行循环TCP、UDP、SCTP转发，用户必须使用apiserver API创建一个服务来配资代理，其实就是kube-proxy通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务访问

* kube-proxy 运行在每个节点上，监听API server中服务对象的变化，再通过管理IPtables或者IPVS规则 来实现网络的转发
* Kube-Proxy不同的版本可支持三种工作模式：
* UserSpace：k8s v1.1之前使用 ， k8s 1.2及以后就已经淘汰
* IPtables：k8s 1.1版本开始支持，1.2开始为默认
* IPVS：k8s 1.9引入到1.11为正式版本，需要安装ipvsadm、ipset工具包和加载ip\_vs内核模块
* IPVS相对IPtables效率会更高一些，使用IPVS模式需要在运行Kube-Proxy的节点上安装ipvsadm、ipset工具包和加载ip\_vs内核模块，当Kube-Proxy以IPVS代理模式启动时，Kube-Proxy将验证节点上是否安装了IPVS模块，如果未安装，则Kube-Proxy将回退到IPtables代理模式。
* 使用IPVS模式，Kube-proxy会监视Kubernetes Services对象和Endpoints，调用宿主机内核Netlink接口以相应地创建IPVS规则并定期与Kubernetes Services对象Endpoints对象同步IPVS规则，以确保IPVS状态与期望一致，访问服务时，流量将被重定向到其中一个后端Pod，IPVS使用哈希表作为底层数据结构并在内核空间中工作，这意味着IPVS可以更快地重定向流量，并且在同步代理规则时具有更好的性能，此外，IPVS为负载均衡算法提供了更多选项，例如：rr（轮训调度） lc（最小连接数）dh（目标哈希） sh（源哈希） sed（最短期望延迟） nq（不排队调度）等。

### 5、kubelet

kubelet是运行在每个worker节点的代理组件，它会监视已分配给节点的pod，具体功能如下：

* 向master汇报node节点的状态信息
* 接受指令并在Pod中创建docker容器
* 准备Pod所需的数据卷
* 返回pod的运行状态
* 在node节点执行容器健康检查

### 6、etcd

etcd是CoreOS公司开发，目前是Kubernetes默认使用的key-value数据存储系统，用于保存kubernetes的所有集群数据，etcd支持分布式集群功能，生产数据使用时需要为etcd数据提供定期备份机制，etcd内部采用raft协议作为一致性算法，etcd基于go语言实现

etcd具有下面这些属性

* 完全复制 集群中的每个节点都可以使用完整的存档
* 高可用性 etcd可用于避免硬件的单点故障或网络问题
* 一致性：每次读取都会返回跨多主机的最新写入
* 简单 包括一个定义良好 面向用户的API(gRPC)
* 安全：实现了带有可选的客户端证书身份验证的自动化TLS
* 快速：每秒10000次写入的基准速度
* 可靠：使用Raft算法实现了存储的合理分布Etcd的工作原理

### 7、DNS

DNS负责为整个集群提供DNS服务，从而实现服务之间的访问

DNS组件历史版本有skydns、kube-dns和coredns三个，k8s 1.3版本之前使用skydns，之后的版本到1.17之间的版本使用kube-dns，目前主要使用coredns，DNS组件用于解析k8s集群中service name所对应得到IP地址



\


