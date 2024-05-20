# Page

### 1、介绍

kubectl 来进行命令行控制k8s集群，原理是调用k8s  apiserver 进行集群操作，apiserver 还包含身份认证、处理请求、对资源增删改查，说到资源包含内置资源和自定义资源CRD，内置资源包含了pod、service、deployment等，这些资源只能通过修改注解和lable来进行修改，操作过程中就需要控制器，controller manager ，控制器包含内置控制器和自定义控制器，另外就是调度器，k8s提供多种调度器来进行资源调度，kubelt 来进行最终的操作，控制CRI、CSI、CNI 来支持容器运行时、存储、网络进行控制。所有控制器都是通过client-go来与API Server进行交互。

2、
