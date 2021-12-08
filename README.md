## 背景
这是一个基于Kubernetes v1.22.2集群部署ELK日志分析系统。并收集与展示Nginx access访问日志的项目。主要创建的原因是由于在测试环境部署的，因此对这个项目用到的K8s相关资源对象yaml文件进行一个存档

## 部署elasticsearch
```
[root@k8s-master ~]# git clone https://github.com/shaxiaozz/Kubernetes-elk.git
[root@k8s-master ~]# cd Kubernetes-elk/Elasticsearch/
[root@k8s-master Elasticsearch]# kubectl apply -f configmap.yaml        #创建configmap资源对象
[root@k8s-master Elasticsearch]# kubectl apply -f service.yaml          #创建service资源对象
[root@k8s-master Elasticsearch]# kubectl apply -f statefulsets.yaml     #创建statefulsets资源对象

[root@k8s-master Elasticsearch]# kubectl get sts,pods,svc,cm -n app | grep elasticsearch
statefulset.apps/elasticsearch   1/1     6d6h
pod/elasticsearch-0                  1/1     Running       0                 4h35m
service/elasticsearch-svc    ClusterIP   None             <none>        9200/TCP,9300/TCP            6d8h
configmap/elasticsearch-cm   1      7d8h
```

## 部署Kibana
```
[root@k8s-master ~]# cd Kubernetes-elk/Kibana/
[root@k8s-master Kibana]# kubectl apply -f configmap.yaml               #创建configmap资源对象
[root@k8s-master Kibana]# kubectl apply -f service.yaml                 #创建service资源对象
[root@k8s-master Kibana]# kubectl apply -f deployment.yaml              #创建deployment资源对象

[root@k8s-master Kibana]# kubectl get deploy,pods,svc,cm -n app | grep kibana
deployment.apps/kibana          1/1     1            1           6d6h
pod/kibana-788586ff48-gdvz5          1/1     Running   0                 6d6h
service/kibana               ClusterIP   10.103.127.77    <none>        5601/TCP                     7d6h
configmap/kibana-cm          1      7d6h
```
## Kibana可视化展示
![nginx](https://user-images.githubusercontent.com/43721571/145190939-f4b758df-2706-4b16-9beb-25d49fce4205.png)

