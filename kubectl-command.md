### 常用的几个kubectl命令

```
kubectl apply -f deploy.yaml ## 基于yaml文件创建k8s资源
kubectl delete -f deploy.yaml ## 基于yaml文件删除k8s资源
kubectl get nodes ## 列出当前k8s有哪些节点
kubectl get ns ## 列出当前集群有哪些namespace
kubectl -n `namespace` get pods ## 列出该namespace下有哪些pods
kubectl -n `namespace` describe pods ## 查看pods的状态和事件
kubectl -n `namespace` logs -f pods ## 查看pods的日志
```
