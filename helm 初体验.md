# helm 初体验

## helm 是什么?

## helm 相比管理yaml有什么优势?

## 以一个demo来试用本地helm

```
  $ wget xxx
  // The helm version you want.
  $ mv xxx  /usr/local/bin/helm
  // Mv bin to /usr/bin etc
  $ helm init
  // Install easily tiller on kubernetes
  $ kubectl get pods -n kube-system |grep tiller
  // To ensure that tiller install successful
```


(demo的helm和tiller均在集群内网中)


## 默认安装的 tiller 权限控制
```
  $ kubectl create serviceaccount --namespace=kube-system tiller

  $ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

  $ kubectl patch deploy --namespace=kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

```
