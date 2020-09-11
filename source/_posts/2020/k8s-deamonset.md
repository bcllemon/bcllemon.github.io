layout: post
title: "K8s DeamonSet"
date: "2020-09-011 10:33"
categories: 技术
---
# 作用
DaemonSet 的主要作用，是让你在 Kubernetes 集群里，运行一个 Daemon Pod。 
# 特征
1. 这个 Pod 运行在 Kubernetes 集群里的每一个节点(Node)上;
2. 每个节点上只有一个这样的 Pod 实例;
3. 当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来;而当旧 节点被删除后，它上面的 Pod 也相应地会被回收掉。

# 场景
1. 各种网络插件的 Agent 组件，都必须运行在每一个节点上，用来处理这个节点上的容器网 络;
2. 各种存储插件的 Agent 组件，也必须运行在每一个节点上，用来在这个节点上挂载远程存 储目录，操作容器的 Volume 目录;
3. 各种监控组件和日志组件，也必须运行在每一个节点上，负责这个节点上的监控信息和日志 搜集。

# 区别
Deployment 部署的副本 Pod 会分布在各个 Node 上，每个 Node 都可能运行好几个副本。DaemonSet 的不同之处在于：每个 Node 上最多只能运行一个副本。

![](/images/2020/k8s-scale.png)
