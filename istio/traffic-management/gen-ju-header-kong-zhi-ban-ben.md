# 根据header控制版本

更改istio-ingress外网访问模式改为nodeport

```
kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'
```

构建两个镜像

{% tabs %}
{% tab title="nginx-v1.yaml" %}
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-demo

---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-demo-deployment-v1
    namespace: default
    labels:
        app: nginx-demo
        version: v1
spec:
    replicas: 2
    selector:
        matchLabels:
            app: nginx-demo
            version: v1
    template:
        metadata:
            labels:
                app: nginx-demo
                version: v1
        spec:
            serviceAccountName: nginx-demo
            containers:
                - name: nginx
                  image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/nginx:v1
                  imagePullPolicy: IfNotPresent
                  ports:
                      - containerPort: 80


---
apiVersion: v1
kind: Service
metadata:
    name: nginx-demo-svc-v1
    namespace: default
spec:
    selector:
        app: nginx-demo
        version: v1
    type: ClusterIP
    ports:
        - name: http
          port: 80
          protocol: TCP
```
{% endtab %}

{% tab title="nginx-v2.yaml" %}
```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-demo-deployment-v2
    labels:
        app: nginx-demo
        version: v2

spec:
    replicas: 2
    selector:
        matchLabels:
            app: nginx-demo
            version: v2
    template:
        metadata:
            labels:
                app: nginx-demo
                version: v2
        spec:
            serviceAccountName: nginx-demo
            containers:
                - name: nginx
                  image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/nginx:v2
                  imagePullPolicy: IfNotPresent
                  ports:
                      - containerPort: 80


---
apiVersion: v1
kind: Service
metadata:
    name: nginx-demo-svc-v2
    namespace: default
spec:
    selector:
        app: nginx-demo
        version: v2
    type: ClusterIP
    ports:
        - name: http
          port: 80
          protocol: TCP
```
{% endtab %}

{% tab title="nginx-demo-svc.yaml" %}
```
apiVersion: v1
kind: Service
metadata:
    name: nginx-demo
    namespace: default
spec:
    selector:
        app: nginx-demo
    type: ClusterIP
    ports:
        - name: http
          port: 80
          protocol: TCP
```
{% endtab %}

{% tab title="" %}
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```
{% endtab %}

{% tab title="" %}
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nginx-demo-virtual-svc
spec:
  hosts:
  - nginx-demo2
  - "nginx-demo.istio-sfkj.sit"

  gateways:
  - nginx-gateway
  http:
    - route:
      - destination:
          host: nginx-demo-svc-v1.default.svc.cluster.local
        weight: 30
      - destination:
          host: nginx-demo-svc-v2.default.svc.cluster.local
        weight: 70

```
{% endtab %}
{% endtabs %}

```
for i in `seq 1 100000`; do wget -q -O - http://nginx-demo-svc && sleep 1;done
```
