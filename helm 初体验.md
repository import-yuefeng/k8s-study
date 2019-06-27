# helm 初体验

## helm 是什么?

## helm 相比管理yaml有什么优势?

## 以一个demo来试用本地helm

(demo的helm和tiller均在集群内网中)


## 默认安装的 tiller 权限控制
```
  $ kubectl create serviceaccount --namespace=kube-system tiller

  $ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

  $ kubectl patch deploy --namespace=kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

```
