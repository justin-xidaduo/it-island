# kiali

#### 安装好demo后给kiali 增加ingress以供外网访问

```
root@pre-web:/nfs_data/k8s_yaml/zhaolibin# kubectl get svc -n istio-system
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
grafana                     ClusterIP      10.103.238.19    <none>        3000/TCP                                                                     39h
istio-egressgateway         ClusterIP      10.107.38.120    <none>        80/TCP,443/TCP,15443/TCP                                                     39h
istio-ingressgateway        LoadBalancer   10.107.25.199    <pending>     15020:31525/TCP,80:32690/TCP,443:32710/TCP,31400:32238/TCP,15443:32761/TCP   39h
istiod                      ClusterIP      10.101.165.45    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP,853/TCP                                40h
jaeger-agent                ClusterIP      None             <none>        5775/UDP,6831/UDP,6832/UDP                                                   39h
jaeger-collector            ClusterIP      10.108.161.116   <none>        14267/TCP,14268/TCP,14250/TCP                                                39h
jaeger-collector-headless   ClusterIP      None             <none>        14250/TCP                                                                    39h
jaeger-query                ClusterIP      10.96.1.213      <none>        16686/TCP                                                                    39h
kiali                       ClusterIP      10.106.194.39    <none>        20001/TCP                                                                    39h
prometheus                  ClusterIP      10.96.218.68     <none>        9090/TCP                                                                     39h
tracing                     ClusterIP      10.104.225.72    <none>        80/TCP                                                                       39h
zipkin                      ClusterIP      10.107.241.143   <none>        9411/TCP                                                                     39h
```

#### 创建ingress yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: istio-kiali-ingress
  namespace: istio-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: ********
    http:
      paths:
      - path: /kiali
        backend:
          serviceName: kiali
          servicePort: 20001
  tls:
  - secretName: smartoffice-pre
```

#### 然后登录http://\*\*\*\*\*\*/kiali

![](<../../.gitbook/assets/image (1).png>)



