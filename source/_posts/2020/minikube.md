layout: post
title: "Macbook Minikube 配置"
date: "2020-09-02 00:00"
categories: 技术
---
# 配置步骤
## 安装 kubectl
``brew install kubernetes-cli``
## 安装 Hypervisor
``brew install hyperkit``
## 安装 Minikube 
``brew install minikube``
## 启动 Minikube 
``minikube start --vm-driver=hyperkit --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers``
## 检查集群的状态
``minikube status``
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
## 查看 Minikube ip
``minikube ip``
## 配置环境变量 
``export no_proxy="127.0.0.1,{minikube ip}"``
## 测试部署
``kubectl create deployment hello-minikube --image=registry.cn-hangzhou.aliyuncs.com/google_containers/echoserver:1.10``
# 将部署暴露为服务
`` kubectl expose deployment hello-minikube --type=NodePort --port=8080``
# 查看服务
``kubectl get pod``
# 获取服务URL
``minikube service hello-minikube --url``
# 查看仪表盘
``minikube dashboard``
# 删除服务
``kubectl delete services hello-minikube``
# 删除部署
``kubectl delete deployment hello-minikube``
# 停止Minikube集群
``minikube stop``
# 删除Minikube集群
``minikube delete``

