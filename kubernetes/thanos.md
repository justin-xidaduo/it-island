# 🥝 thanos

Thanos 是一个用于 Kubernetes 集群监控的高可用、可扩展的 Prometheus 扩展，它能够提供多集群、多区域的监控数据的聚合、查询和存储功能。Thanos 主要解决了 Prometheus 在扩展性、高可用性和跨集群查询方面的限制。

在 Kubernetes 中使用 Thanos 主要有以下几个组件：

#### 1. **Thanos Query**

* 提供全局的查询接口，允许用户查询不同 Prometheus 实例和 Thanos 存储中聚合的数据。
* 它允许跨多个集群、区域进行查询，统一呈现 Prometheus 数据。

#### 2. **Thanos Store**

* 存储 Prometheus 数据的长期存储组件。它将历史时间序列数据从本地 Prometheus 存储迁移到对象存储（如 S3、GCS、Azure Blob 等），实现长期存储。
* 提供查询历史数据的能力，通常与 Thanos Query 配合使用。

#### 3. **Thanos Sidecar**

* 它与 Prometheus 配合使用，作为一个 sidecar 容器运行在同一个 Pod 内。
* 它将 Prometheus 的数据上传到远程对象存储，并将 Prometheus 的元数据暴露给 Thanos Query 进行跨集群查询。

#### 4. **Thanos Compact**

* 负责将来自多个 Thanos Store 的数据合并，并对其进行压缩。它是一个数据处理组件，帮助你对存储数据进行优化。

#### 5. **Thanos Receive**

* 接收来自 Prometheus 或其他数据源的时间序列数据，支持写入 Thanos 存储后端。

#### Thanos 部署模式

1. **单集群部署**：
   * 在单个 Kubernetes 集群内，你可以运行一个 Prometheus 实例，并部署 Thanos Sidecar 与其配合，配合 Thanos Query 和 Store 来实现全局查询和数据存储。
2. **多集群部署**：
   * 在多个集群中，你可以分别部署 Prometheus 和 Thanos Sidecar，然后将多个 Thanos Store 聚合在一起，使用 Thanos Query 来进行跨集群的查询。

#### 在 Kubernetes 中安装 Thanos

可以使用 Helm charts 或者 Kubernetes manifests 来部署 Thanos。

**示例：使用 Helm 部署 Thanos**

```bash
bash复制编辑# 添加 Thanos Helm 仓库
helm repo add thanos https://thanos-io.github.io/thanos

# 更新 Helm 仓库
helm repo update

# 部署 Thanos
helm install thanos thanos/thanos --namespace monitoring
```

这会自动部署 Thanos 的各个组件，包括 Query、Sidecar、Store 等。

#### 配置 Prometheus 与 Thanos 集成

1. **部署 Prometheus 与 Thanos Sidecar**：
   * 在 Prometheus 配置中添加 Thanos Sidecar，确保其能够上传数据到对象存储，并通过 Thanos Store 进行数据查询。
2. **跨集群配置**：
   * 在多集群环境中，可以使用 Thanos Query 来进行全局查询，配置每个集群的 Thanos Store，使得查询数据可以跨集群进行。

#### Thanos 的优点

* **高可用性**：通过使用多个 Prometheus 实例和 Thanos，避免了单点故障。
* **跨集群查询**：可以查询来自多个 Kubernetes 集群的数据，支持全局视图。
* **长期存储**：Thanos 可以将 Prometheus 的数据存储在对象存储中，有效解决了 Prometheus 存储限制的问题。

如果你已经在使用 Prometheus 来监控 Kubernetes 集群，Thanos 可以让你更轻松地管理跨集群的数据，提升你的监控系统的可扩展性和高可用性。
