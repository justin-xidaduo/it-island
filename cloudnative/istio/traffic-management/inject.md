# inject

### 开启inject

只要在namespace lable 上增加istio-injection=enabled

```bash
kubectl label namespaces default istio-injection=enabled
# namespace/default labeled
kubectl describe namespaces default
# Name:         default
# Labels:       field.cattle.io/projectId=p-5phwk
#               istio-injection=enabled
# Annotations:  cattle.io/status:
#                 {"Conditions":[{"Type":"ResourceQuotaInit","Status":"True","Message":"","LastUpdateTime":"2020-06-22T13:16:03Z"},{"Type":"InitialRolesPopu...
#              field.cattle.io/projectId: local:p-5phwk
#              lifecycle.cattle.io/create.namespace-auth: true
# Status:       Active

# No resource quota.

# No LimitRange resource.
```

### 关闭inject

```bash
kubectl label namespaces default istio-injection-
# namespace/default labeled
```
