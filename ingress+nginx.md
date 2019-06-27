# ingress+nginx

ingress+nginx 指南, 当你部署 k8s 集群在一个不支持 Kubernetes Engine 的云厂商上时(例如小的厂商), 此时在将你的服务透明地暴露公网访问时, 
你可能无法使用 k8s 提供的 **Load Balancer** 服务. 此时你需要使用 Ingress Controller + Ingress 的解决办法来实现这个目的.

本文旨在记录一次部署 Ingress Controller + Ingress 的历程(Ingress Controller: Nginx)

## 使得k8s服务公网访问的办法有哪些?

要了解k8s服务暴露公网访问的办法前, 请先明确多个k8s内的资源(resource)概念.

#### pod

关于pod的描述参考 [掘金](https://juejin.im/post/5b6cef7851882536054a812b)

Pod (容器组)，英文中 Pod 是豆荚的意思。从名字的含义可以看出，Pod 是一组有依赖关系的容器。Pod 是 Kubernetes 集群中最基本的资源对象，每个 Pod 由一个或多个业务容器和一个根容器 (Pause 容器) 组成。Kubernetes 为每个 Pod 分配了唯一的 IP（即：Pod IP），Pod 里的多个容器共享这个 IP。Pod 内的容器除了 IP，还共享相同的网络命名空间、端口、存储卷等，也就是说这些容器之间能通过 localhost 来通信。Pod 包含的容器都会运行在同一个节点上，也可以同时启动多个相同的 Pod 用于 Failover 或者 Load balance。

一个pod中主要放置关系耦合的相关容器组, 相当于一个"篮子"。

注意! Pod 的生命周期是短暂的，Kubernetes 会根据应用的配置对 Pod 进行创建、销毁并根据监控指标进行伸缩扩容。当pod工作不正常时, k8s会尝试将pod销毁, 并且重建新的来替代。Kubernetes 在创建 Pod 时可以选择集群中的任何一台空闲的节点上进行，因此其网络地址是不固定的。

* 由于 Pod 的这一特点，一般不建议直接通过 Pod 的地址去访问应用。*
那么对于除pod外的外部服务如何稳定地访问到相关服务呢? k8s引入了一个新的概念, 叫做service.

#### service

关于service的描述参考 [掘金](https://juejin.im/post/5b6cef7851882536054a812b)

为了解决访问 Pod 不方便直接访问的问题，Kubernetes 采用了 Service 对 Pod 进行封装。

Service 是对后端提供服务的一组 Pod 的抽象，Service 会绑定到一个固定的虚拟 IP上(Cluster IP)。

该虚拟 IP 只在 Kubernetes Cluster 中可见，但其实该虚拟 IP 并不对应一个虚拟或者物理设备，而只是 IPtables 中的规则，然后再通过 IPtables 将服务请求路由到后端的 Pod 中。通过这种方式，可以确保服务消费者可以稳定地访问 Pod 提供的服务，而不用关心 Pod 的创建、删除、迁移等变化以及如何用一组 Pod 来进行负载均衡。

实现 Service 这一功能的关键是由 Kubernetes 中的 Kube-Proxy 来完成的。Kube-Proxy 运行在每个节点上，监听 k8s apiserver 中服务对象的变化，再通过管理 IPtables 来实现网络的转发。Kube-Proxy 目前支持三种模式：UserSpace、IPtables、IPVS。

### 那么让服务暴露就应该是针对于service resource
关于服务暴露的描述参考 [kubernetes-handbook](https://github.com/rootsongjc/kubernetes-handbook/)

通过上述介绍, 我们需要将某个服务暴露公网也就需要针对service来进行.
在当前k8s版本下(1.14), 一共有如下几种办法:

1. NodePort
2. LoadBalancer
3. Ingress

## NodePort模式

其中, 最容易部署和理解的是第一种, 也就是NodePort模式, 这种模式下你只需要对需要进行暴露的service的yaml/json 描述文件 进行小小的改动即可

以下为一个service yaml 的例子:

** dashboard-clusterIP.yaml **

---------------------------------------------------------
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: ClusterIP
 ```
---------------------------------------------------------
当前服务所定义的网络类型为ClusterIP, 当你没有填写spec.type 字段(缺省)或者该字段的值为ClusterIP, 该服务的网络类型均为Cluster IP.
该模式下可以通过clusterIP:servicePort来访问到服务, 一般为集群内可访问, 集群外不可访问.

接下来我们把它改为NodePort模式:

** dashboard-NodePort.yaml **

---------------------------------------------------------
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
 ```
---------------------------------------------------------
你可以看到, 实际上我们就修改了一个字段, type即可实现. 那么你会问, 当改动了该字段后, k8s会为该service提供的哪一个port呢?
实际上, NodePort所分配的端口可以自行指定, 如果如上文般缺省, 那么将会被默认分配到宿主机的30000-32767可用端口上. 
但是你也可以改变该范围, 这个值在API server的配置文件中，用--service-node-port-range定义。

如果你要确定指定一个外部port可以通过增加 spec.ports.port.nodePort 参数来实现:

** dashboard-NodePort-configNodePort.yaml **

---------------------------------------------------------
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
 ```
---------------------------------------------------------

## Loadbalancer模式


## ingress 是什么?


## 目前有什么ingress controller方案?

## 在k8s集群中实现ingress服务(以部署一个nginx service为例)

