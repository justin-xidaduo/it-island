---
description: RequestLimit
---

# 🍑 Request Limit

## 1、所有容器都应该设置 request

request 的值并不是指给容器实际分配的资源大小，它仅仅是给调度器看的，调度器会 "观察" 每个节点可以用于分配的资源有多少，也知道每个节点已经被分配了多少资源。被分配资源的大小就是节点上所有 Pod 中定义的容器 request 之和，它可以计算出节点剩余多少资源可以被分配(可分配资源减去已分配的 request 之和)。如果发现节点剩余可分配资源大小比当前要被调度的 Pod 的 reuqest 还小，那么就不会考虑调度到这个节点，反之，才可能调度。所以，如果不配置 request，那么调度器就不能知道节点大概被分配了多少资源出去，调度器得不到准确信息，也就无法做出合理的调度决策，很容易造成调度不合理，有些节点可能很闲，而有些节点可能很忙，甚至 NotReady。

所以，建议是给所有容器都设置 request，让调度器感知节点有多少资源被分配了，以便做出合理的调度决策，让集群节点的资源能够被合理的分配使用，避免陷入资源分配不均导致一些意外发生。

## 2、经常忘记设置怎么办

有时候我们会忘记给部分容器设置 request 与 limit，其实我们可以使用 LimitRange 来设置 namespace 的默认 request 与 limit 值，同时它也可以用来限制最小和最大的 request 与 limit。 示例:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: test
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 100m
    type: Container
```

## 3、重要的应用如何配置

节点资源不足时，会触发自动驱逐，将一些低优先级的 Pod 删除掉以释放资源让节点自愈。没有设置 request，limit 的 Pod 优先级最低，容易被驱逐；request 不等于 limit 的其次； request 等于 limit 的 Pod 优先级较高，不容易被驱逐。所以如果是重要的线上应用，不希望在节点故障时被驱逐导致线上业务受影响，就建议将 request 和 limit 设成一致。

## 4、为什么 CPU 利用率远不到 limit 还会被 throttle ? <a href="#wei-shen-me-cpu-li-yong-lv-yuan-bu-dao-limit-hai-hui-bei-throttle" id="wei-shen-me-cpu-li-yong-lv-yuan-bu-dao-limit-hai-hui-bei-throttle"></a>

CPU 限流是因为内核使用 CFS 调度算法，对于微突发场景，在一个 CPU 调度周期内 (100ms) 所占用的时间超过了 limit 还没执行完，就会强制 "抢走" CPU 使用权(throttle)，等待下一个周期再执行，但是时间拉长一点，进程使用 CPU 所占用的时间比例却很低，监控上就看不出来 CPU 有突增，但实际上又被 throttle 了。

更多详细解释参考 [k8s CPU limit和throttling的迷思](https://zhuanlan.zhihu.com/p/433065108)。



###
