# demo

#### 安装dem

```
istioctl manifest apply --set profile=demo
```

#### 卸载demo

```
istioctl manifest generate --set profile=demo | kubectl delete -f -
```
