layout: post
title: "K8s 水平扩展"
date: "2020-09-04 17:02"
categories: 技术
---
# Deployment 关系
Deployment 通过“控制器模式”，来操作 ReplicaSet 的个数和属性，进 而实现“水平扩展 / 收缩”和“滚动更新”这两个编排动作。
![](/images/2020/k8s-scale.png)
# 增加节点
``
kubectl scale deployment nginx-deployment --replicas=4
``
# 查看扩展状态
``
kubectl rollout status deployment/nginx-deployment
``
# 修改配置
``
kubectl edit deployment/nginx-deployment
``
# 回滚
``
kubectl rollout undo deployment/nginx-deployment
``
# 变更历史 
``
kubectl apply -f nginx-deployment2.yaml --record
kubectl rollout history deployment/nginx-deployment
``
# 查看版本变更
``
kubectl rollout history deployment/nginx-deployment --revision=2
``
# 回滚指定版本
``
kubectl rollout undo deployment/nginx-deployment --to-revision=2
``
# 暂停滚动更新
``
kubectl rollout pause deployment/nginx-deployment
``
# 恢复滚动更新
``
kubectl rollout resume deployment/nginx-deployment
``
