# Ingress

## Ingress 简介

### 理解Ingress

ingress就是从kubernetes集群外访问集群的入口，将用户的URL请求转发到不同的service上。Ingress相当于nginx、apache等负载均衡方向代理服务器，其中还包括规则定义，即URL的路由信息，路由信息得的刷新由[Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers)来提供。

### 理解Ingress controller

Ingress Controller 实质上可以理解为是个监视器，Ingress Controller 通过不断地跟 kubernetes API 打交道，实时的感知后端 service、pod 等变化，比如新增和减少 pod，service 增加与减少等；当得到这些变化信息后，Ingress Controller 再结合下文的 Ingress 生成配置，然后更新反向代理负载均衡器，并刷新其配置，达到服务发现的作用。

## Ingress controller&#x20;

### Ingress controller 对比

![](../../.gitbook/assets/ingress\_controller.jpg)

### Traefik

#### 介绍traefik

[Traefik](https://traefik.io/)是一款开源的反向代理与负载均衡工具。它最大的优点是能够与常见的微服务系统直接整合，可以实现自动化动态配置。目前支持Docker, Swarm, Mesos/Marathon, Mesos, Kubernetes, Consul, Etcd, Zookeeper, BoltDB, Rest API等等后端模型。

#### 架构图

[https://docs.traefik.io/assets/img/traefik-architecture.png](https://docs.traefik.io/assets/img/traefik-architecture.png)

### Nginx Ingress

#### 介绍

[NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) 是ingress controller通过和kubernetes api交互，动态的去感知集群中ingress规则变化，然后读取它，按照自定义的规则，规则就是写明了哪个域名对应哪个service，生成一段nginx配置，再写到nginx-ingress-control的pod里，这个Ingress controller的pod里运行着一个Nginx服务，控制器会把生成的nginx配置写入/etc/nginx.conf文件中，然后reload一下使配置生效。以此达到域名分配置和动态更新的问题。

#### 参考链接:

* [**Kubernetes实战：集群中部署NGINX Ingress Controller**](https://www.jianshu.com/p/613967aee68e)
* [**浅谈 k8s ingress controller 选型**](https://zhuanlan.zhihu.com/p/109458069)
