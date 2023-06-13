---
description: 官网https://rancher.com/
---

# Rancher

#### [官方文档](https://rancher.com/docs/rancher/latest/en/)&#x20;

## 简介

Rancher是一个开源软件平台，使组织能够在生产中运行和管理Docker和Kubernetes。使用Rancher，组织不再需要使用一套独特的开源技术从头开始构建容器服务平台。Rancher提供了管理生产中的容器所需的整个软件堆栈。

## 主要组件

### INFRASTRUCTURE ORCHESTRATION  基础架构流程

Rancher以Linux主机的形式从任何公共或私有云中获取原始计算资源。每个Linux主机可以是虚拟机或物理机。Rancher对每个主机的期望不超过CPU，内存，本地磁盘存储和网络连接。从Rancher的角度来看，来自云提供商的VM实例和托管在云设施中的裸机服务器是无法区分的。

Rancher实现了可移植的基础结构服务层，专门为容器化应用程序提供动力。Rancher基础架构服务包括网络，存储，负载平衡器，DNS和安全性。Rancher基础结构服务通常本身是作为容器部署的，因此同一Rancher基础结构服务可以在来自任何云的任何Linux主机上运行。

### 容器编排和调度  CONTAINER ORCHESTRATION AND SCHEDULING

许多用户选择使用容器编排和调度框架来运行容器化的应用程序。Rancher分发了当今所有流行的容器编排和调度框架，包括Docker Swarm，Kubernetes和Mesos。同一用户可以创建多个Swarm或Kubernetes集群。然后，他们可以使用本机Swarm或Kubernetes工具来管理其应用程序。

除了Swarm，Kubernetes和Mesos外，Rancher还支持其自己的称为Cattle的容器编排和调度框架。Rancher广泛使用Cattle来协调基础架构服务以及设置，管理和升级Swarm，Kubernetes和Mesos集群。

### 应用目录 APPLICATION CATALOG

Rancher用户只需单击一下按钮，即可从应用程序目录中部署整个多容器集群应用程序。当新版本的应用程序可用时，用户可以管理已部署的应用程序并执行全自动升级。Rancher维护由Rancher社区贡献的流行应用程序组成的公共目录。Rancher用户可以创建自己的私有目录。

### 企业级控制 ENTERPRISE-GRADE CONTROL

Rancher支持灵活的用户身份验证插件，并与Active Directory，LDAP和GitHub进行了预先构建的用户身份验证集成。Rancher在环境级别支持基于角色的访问控制（RBAC），允许用户和组共享或拒绝对开发和生产环境的访问。

### Rancher的主要组件和功能。

![](https://rancher.com/docs/one-point-x/img/rancher/rancher\_overview\_2.png)

## 为什么使用rancher？

**Rancher是供采用容器的团队使用的完整软件堆栈。它解决了在任何基础架构上管理多个Kubernetes集群的运营和安全挑战，同时为DevOps团队提供了用于运行容器化工作负载的集成工具**。\
用户不需要深入了解kubernetes概念就可以使用rancher，rancher包含应用商店，支持一键部署helm和compose模板。rancher通过各种云、本地生态系统产品认证，其中包括安全工具，监控系统，容器仓库以及存储和网络驱动程序，下图说明rancher在IT和DevOps组织中扮演的角色，每个团队都会在他们选择的公共云或私有云上部署应用

### 主要功能

* 内置CI/CD；&#x20;
* 告警和日志收集；&#x20;
* 多集群管理；&#x20;
* rancher kubernetes engine（RKE）;&#x20;
* 与云Kubernetes服务（如GKE,EKS和AKS）集成；
