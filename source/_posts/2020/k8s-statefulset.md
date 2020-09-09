layout: post
title: "K8s StatefulSet"
date: "2020-09-09 17:03"
categories: 技术
---
# Stateful Application
实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态 应用”(Stateful Application)
# 拓扑状态
这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必 须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到 这个新 Pod。
## StatefulSet实现
StatefulSet 这个控制器的主要作用之一，就是使用 Pod 模板创建 Pod 的时候， 对它们进行编号，并且按照编号顺序逐一完成创建工作。而当 StatefulSet 的“控 制循环”发现 Pod 的“实际状态”与“期望状态”不一致，需要新建或者删除 Pod 进行“调谐”的时候，它会严格按照这些 Pod 编号的顺序，逐一完成这些操作。通过 Headless Service 的方式，StatefulSet 为每个 Pod 创建了一个固定并且稳定 的 DNS 记录，来作为它的访问入口。
## Headless Service 配置
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None
```
## StatefulSet 配置
```
apiVersion: apps/v1 
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.8
          ports:
            - containerPort: 80
              name: web
```
# 存储状态
这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实 例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一 份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的 多个存储实例。
## 实现
Kubernetes 项目引入了一组叫作 Persistent Volume Claim(PVC)和 Persistent Volume(PV)的 API 对象. Kubernetes 中 PVC 和 PV 的设计，实际上类似于“接口”和“实现”的思想。开发者 只要知道并会使用“接口”，即:PVC;而运维人员则负责给“接口”绑定具体的实现，即: PV。而 PVC、PV 的设计，也使得 StatefulSet 对存储状态的管理成为了可能。
StatefulSet 为每一个 Pod 分配并创建一个同样编号的 PVC。这样，Kubernetes 就可 以通过 Persistent Volume 机制为这个 PVC 绑定上对应的 PV，从而保证了每一个 Pod 都拥有 一个独立的 Volume。
## PVC 配置
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
## StatefulSet 配置
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```
# 总结
StatefulSet 其实就是一种特殊的 Deployment，而其独特之处在于，它的每个 Pod 都被编号了。而且，这个编号会体现在 Pod 的名字和 hostname 等标识信息上，这不仅代表了 Pod 的创建顺序，也是 Pod 的重要网络标 识(即:在整个集群里唯一的、可被的访问身份)。
