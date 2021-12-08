## 一、背景
这是一个基于Kubernetes v1.22.2集群部署ELK日志分析系统。并收集与展示Nginx access访问日志的项目。主要创建的原因是由于在测试环境部署的，因此对这个项目用到的K8s相关资源对象yaml文件进行一个存档

## 二、部署elasticsearch
```
[root@k8s-master ~]# git clone https://github.com/shaxiaozz/Kubernetes-elk.git
[root@k8s-master ~]# cd Kubernetes-elk/Elasticsearch/
[root@k8s-master Elasticsearch]# kubectl apply -f configmap.yaml    #创建configmap资源对象
[root@k8s-master Elasticsearch]# kubectl apply -f service.yaml      #创建service资源对象
[root@k8s-master Elasticsearch]# kubectl apply -f statefulsets.yaml #创建statefulsets资源对象
```
## Kibana可视化展示
![nginx](https://user-images.githubusercontent.com/43721571/145190939-f4b758df-2706-4b16-9beb-25d49fce4205.png)

