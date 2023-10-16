# 其他

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

\




