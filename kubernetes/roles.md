# roles

#### 查询k8s所有node节点

```
[root@k8s-node001 ~]# kubectl get node
NAME          STATUS   ROLES    AGE     VERSION
k8s-node001   Ready    master   3m20s   v1.18.2
k8s-node002   Ready    <none>   2m25s   v1.18.2
```

#### 设置k8s-node001为master角色(现已经是master，若不是，设置命令如下)

```
kubectl label nodes k8s-node001 node-role.kubernetes.io/master=
```

#### 设置master 一般情况下不接受负载

```
kubectl taint nodes k8s-node001 node-role.kubernetes.io/master=true:NoSchedule
```

#### 设置k8s-node002为node角色

```
kubectl label nodes k8s-node002 node-role.kubernetes.io/node=
```

#### 设置k8s-node001即是master节点又是node节点，并接受负载调度，运行pod

```
# 增加角色为node
kubectl label nodes k8s-node001 node-role.kubernetes.io/node=
# 查看结果
kubectl get node
# NAME          STATUS   ROLES         AGE   VERSION
# k8s-node001   Ready    master,node   12m   v1.18.2
# k8s-node002   Ready    node          11m   v1.18.2

# 设置k8s-node001接受负载调度，运行pod
kubectl taint node k8s-node001 node-role.kubernetes.io/master-

# 测试 
[root@k8s-node001 ~]# kubectl apply -f /data/kubernetes/ingress/tradfik/nginx-demo.yaml
deployment.apps/nginx-demo-deployment created
service/nginx-demo-svc created
ingress.extensions/nginx-demo-ingress created
[root@k8s-node001 ~]# kubectl get pod -n default
NAME                                     READY   STATUS    RESTARTS   AGE
nginx-demo-deployment-64ddcbbffb-dfx6f   1/1     Running   0          8s
nginx-demo-deployment-64ddcbbffb-h4gh7   1/1     Running   0          8s
nginx-demo-deployment-64ddcbbffb-h6qss   1/1     Running   0          8s
nginx-demo-deployment-64ddcbbffb-t9cth   1/1     Running   0          8s
nginx-demo-deployment-64ddcbbffb-zdvrz   1/1     Running   0          8s
[root@k8s-node001 ~]# kubectl get pod -n default -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-demo-deployment-64ddcbbffb-dfx6f   1/1     Running   0          12s   10.244.1.6   k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-h4gh7   1/1     Running   0          12s   10.244.0.5   k8s-node001   <none>           <none>
nginx-demo-deployment-64ddcbbffb-h6qss   1/1     Running   0          12s   10.244.1.5   k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-t9cth   1/1     Running   0          12s   10.244.0.4   k8s-node001   <none>           <none>
nginx-demo-deployment-64ddcbbffb-zdvrz   1/1     Running   0          12s   10.244.1.4   k8s-node002   <none>           <none>
```

#### 设置master不运行pod

```
kubectl taint nodes test1 node-role.kubernetes.io/master=:NoSchedule

# 测试
[root@k8s-node001 ~]# kubectl delete -f /data/kubernetes/ingress/tradfik/nginx-demo.yaml
deployment.apps "nginx-demo-deployment" deleted
service "nginx-demo-svc" deleted
ingress.extensions "nginx-demo-ingress" deleted
[root@k8s-node001 ~]# kubectl apply -f /data/kubernetes/ingress/tradfik/nginx-demo.yaml
[root@k8s-node001 ~]# kubectl get pod -n default -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
nginx-demo-deployment-64ddcbbffb-k5v8f   1/1     Running   0          6s    10.244.1.8    k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-kmr79   1/1     Running   0          6s    10.244.1.10   k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-m98m2   1/1     Running   0          6s    10.244.1.7    k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-s9qnw   1/1     Running   0          6s    10.244.1.9    k8s-node002   <none>           <none>
nginx-demo-deployment-64ddcbbffb-x2297   1/1     Running   0          6s    10.244.1.11   k8s-node002   <none>           <none>
```
