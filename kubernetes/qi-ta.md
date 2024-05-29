# 🥭 其他

### 1、deployment的更新升级策略 <a href="#deployment-de-geng-xin-sheng-ji-ce-lve-you-na-xie" id="deployment-de-geng-xin-sheng-ji-ce-lve-you-na-xie"></a>

* Recreate 重建更新：这种更新策略会杀掉所有正在运行的pod，然后再重新创建的pod；
* rollingUpdate 滚动更新：这种更新策略，deployment会以滚动更新的方式来逐个更新pod，同时通过设置滚动更新的两个参数`maxUnavailable、maxSurge`来控制更新的过程。

### 2、deployment的滚动更新策略有两个特别主要的参数maxUnavailable、maxSurge。

* maxUnavailable：最大不可用数，maxUnavailable用于指定deployment在更新的过程中不可用状态的pod的最大数量，maxUnavailable的值可以是一个整数值，也可以是pod期望副本的百分比，如25%，计算时向下取整。
* maxSurge：最大激增数，maxSurge指定deployment在更新的过程中pod的总数量最大能超过pod副本数多少个，maxUnavailable的值可以是一个整数值，也可以是pod期望副本的百分比，如25%，计算时向上取整。

### 3、kube-proxy原理

集群中每个Node上都会运行一个kube-proxy服务进程，他是Service的透明代理兼均衡负载器，其核心功能是将某个Service的访问转发到后端的多个Pod上。kube-proxy通过监听集群状态变更，并对本机iptables做修改，从而实现网络路由。而其中的负载均衡，也是通过iptables的特性实现的。

从V1.8版本开始，用IPVS（IP Virtual Server）模式，用于路由规则的配置，主要优势是：

1）为大型集群提供了更好的扩展性和性能。采用哈希表的数据结构，更高效；

2）支持更复杂的负载均衡算法；

3）支持服务器健康检查和连接重试；

4）可以动态修改ipset的集合；

### 4、Qos 服务质量等级

QoS（Quality of Service），大部分译为“服务质量等级”，又译作“服务质量保证”，是作用在 Pod 上的一个配置，当 Kubernetes 创建一个 Pod 时，它就会给这个 Pod 分配一个 QoS 等级，可以是以下等级之一：

* **Guaranteed**：Pod 里的每个容器都必须有内存/CPU 限制和请求，而且值必须相等。
* **Burstable**：Pod 里至少有一个容器有内存或者 CPU 请求且不满足 Guarantee 等级的要求，即内存/CPU 的值设置的不同。
* **BestEffort**：容器必须没有任何内存或者 CPU 的限制或请求。

该配置不是通过一个配置项来配置的，而是通过配置 CPU/内存的 `limits` 与 `requests` 值的大小来确认服务质量等级的。使用 `kubectl get pod -o yaml` 可以看到 pod 的配置输出中有 `qosClass` 一项。该配置的作用是为了给资源调度提供策略支持，调度算法根据不同的服务质量等级可以确定将 pod 调度到哪些节点上。

Kubernetes 在 Node 资源不足时使用 QoS 类来就驱逐 Pod 作出决定。

\




