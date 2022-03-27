# 基本操作

```
# 查看node信息
kubectl get nodes

# 查看所有pod信息
kubectl get pod --all-namespaces -o wide
kubectl get pods

# 查看当前所有service
kubectl get services

# pod执行命令
kubectl exec nginx-6k6q6 ls

# pod查看日志
kubectl logs nginx-6k6q6

# 删除某个pod
kubectl delete pod kubernetes-dashboard-7448ffc97b-lrlnt --namespace=kubernetes-dashboard
kubectl delete pod $i --force --grace-period=0 -n base-service
# 推荐使用方式
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -

# 删除单个 web名字的pods
kubectl delete deployment web

# 删除单个容器
kubectl delete svc web1

##### K8S踢出集群
kubectl drain 192.168.100.82 --delete-local-data --force --ignore-daemonsets
kubectl delete node 192.168.100.82

##### 导出yaml文件
kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml

# port为容器端口 target-port为暴露端口 type为类型
kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1 -o yaml > web1.yaml

# 给pod添加多个
kubectl scale deployment web --replicas=5

#应用升级回滚和弹性伸缩
kubectl set image deployment web nginx=nginx:1.15 //升级应用版本
kubectl rollout status deployment web             // 查看升级状态
kubectl rollout history deployment web            // 查看升级的版本
kubectl rollout undo deployment web               // 回滚上一个版本
kubectl rollout undo deployment web --to-revision=2  // 回滚到指定的版本
kubectl scale deployment web --replicas=10        //弹性伸缩

kubectl get componentstatuses //查看node节点组件状态
kubectl get svc -n kube-system //查看应用
kubectl cluster-info //查看集群信息
kubectl describe --namespace kube-system service kubernetes-dashboard //详细服务信息
kubectl apply -f kube-apiserver.yaml //更新kube-apiserver容器
kubectl delete -f /root/k8s/k8s_images/kubernetes-dashboard.yaml //删除应用
kubectl delete service example-server //删除服务
systemctl start kube-apiserver.service //启动服务。
kubectl get deployment --all-namespaces //启动的应用
kubectl get pod -o wide --all-namespaces //查看pod上跑哪些服务
kubectl delete pods <pod>        // 删除pods
kubectl get pod -o wide -n kube-system //查看应用在哪个node上
kubectl describe pod --namespace=kube-system //查看pod上活动信息
kubectl describe depoly kubernetes-dashboard -n kube-system
kubectl get depoly kubernetes-dashboard -n kube-system -o yaml
kubectl get service kubernetes-dashboard -n kube-system //查看应用
kubectl delete -f kubernetes-dashboard.yaml //删除应用
kubectl get events //查看事件
kubectl get rc/kubectl get svc
kubectl get namespace //获取namespace信息
kubectl delete node 节点名 //删除节点
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}') // 获取令牌
```
