# ingress+nginx

ingress+nginx 指南, 当你部署 k8s 集群在一个不支持 Kubernetes Engine 的云厂商上时(例如小的厂商), 此时在将你的服务透明地暴露公网访问时, 
你可以无法使用 k8s 提供的 **Load Balancer** 服务. 此时你需要使用 Ingress Controller + Ingress 的解决办法来实现这个目的.

本文旨在记录一次部署 Ingress Controller + Ingress 的历程(Ingress Controller: Nginx)

## 使得k8s服务公网访问的办法有哪些?

## ingress 是什么?

## 目前有什么ingress controller方案?

## 在k8s集群中实现ingress服务(以部署一个nginx service为例)

