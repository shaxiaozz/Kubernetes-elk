## 目录
- [背景](#背景)
- [部署elasticsearch](#部署elasticsearch)
- [部署Kibana](#部署Kibana)
- [部署Zookeeper](#部署Zookeeper)
- [部署Kafka](#部署Kafka)
- [部署Logstash](#部署Logstash)
- [安装及配置filebeat](#安装及配置filebeat)
- [配置Nginx转发Kibana](#配置Nginx转发Kibana)
- [Kibana可视化展示](#Kibana可视化展示)

## 背景
这是一个基于Kubernetes v1.22.2集群部署ELK日志分析系统。并收集与展示Nginx access访问日志的项目。主要创建的原因是由于在测试环境部署的，因此对这个项目用到的K8s相关资源对象yaml文件进行一个存档

请根据目录的顺序来部署各个服务。由于我的测试环境资源有限，每个服务都只是运行了一个副本(Pod)。因此，如果你想针对ES，Zookeeper，Kafka服务做集群的话，也是可以的，只需要修改控制器的副本数与对于服务的configmap配置文件即可。

**以下是架构图**
![Kubernetes-elk架构 (1)](https://user-images.githubusercontent.com/43721571/145380271-4dd00308-4ce0-458a-96e8-9bd480575523.png)

1、Filebeat组件为日志采集客户端：通过定义服务器的日志文件路径，把日志一行一行的传输给到Kafka集群。因此在Filebeat配置文件中，input输入即是服务器日志文件的路径，output输出即是kafka连接的地址与设置kafka的topic。

2、对于Kafka集群来说，Filebeat输出的日志到它哪里就作为生产者了，而Logstash就会把Filebeat生产到的信息给消息了，因此对于Kafka来说，Logstash就是消费组了，而logstash的消费组就叫logstash。

3、Logstash消费Kafka消息时，通过判断对于的topic名称来决定是否需要进行日志的切割与过滤。在完成日志的切割与过滤后，将把日志流输出到ES上，进行存储。因此对于Logstash来说，input输入就是Kafka topic里面的消息，output输出就是ES的索引。

4、当Logstash输出日志到达ES后，ES还根据配置来对日志进行存储，存储的索引是什么，分片是多少，副本数是多少等等。至此，日志的收集，过滤，存储已经完成了。

5、当用户需要查看日志时，首先通过外网域名(kibana.qiufeng5.cn)访问，流量到达Nginx网关后，Nginx网关通过代理规则，将流量转发到了后端的Kibana实例上，Kibana再去连接ES集群，然后再返回结果给到用户。

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
endpoints/elasticsearch-svc    192.168.2.68:9200,192.168.2.68:9300                     6d8h
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
endpoints/kibana               192.168.3.30:5601                                       7d6h
configmap/kibana-cm          1      7d6h
```

## 部署Zookeeper
```
[root@k8s-master ~]# cd Kubernetes-elk/Zookeeper/
[root@k8s-master Zookeeper]# kubectl apply -f configmap.yaml            #创建configmap资源对象
[root@k8s-master Zookeeper]# kubectl apply -f service.yaml              #创建service资源对象
[root@k8s-master Zookeeper]# kubectl apply -f statefulsets.yaml         #创建statefulsets资源对象

[root@k8s-master Zookeeper]# kubectl get sts,pods,svc,ep,cm -n app | grep zookeeper
statefulset.apps/zookeeper       1/1     12h
pod/zookeeper-0                      1/1     Running   0                 12h
service/zookeeper-svc        ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP   12h
endpoints/zookeeper-svc        192.168.5.30:2181,192.168.5.30:2888,192.168.5.30:3888   12h
configmap/zookeeper-cm       1      5d9h
```

## 部署Kafka
```
[root@k8s-master ~]# cd Kubernetes-elk/Kafka/
[root@k8s-master Kafka]# kubectl apply -f service-node.yaml             #创建service资源对象（有ClusterIP）
[root@k8s-master Kafka]# kubectl apply -f service.yaml                  #创建service资源对象（无ClusterIP）
[root@k8s-master Kafka]# kubectl apply -f statefulsets.yaml             #创建statefulsets资源对象    

[root@k8s-master Kafka]# kubectl get sts,pods,svc,ep,cm -n app | grep kafka
statefulset.apps/kafka           1/1     12h
pod/kafka-0                          1/1     Running   0                 12h
service/kafka-svc            ClusterIP   None             <none>        9092/TCP                     12h
service/kafka-svc-node       ClusterIP   10.110.177.58    <none>        9092/TCP                     12h
endpoints/kafka-svc            192.168.5.31:9092                                       12h
endpoints/kafka-svc-node       192.168.5.31:9092                                       12h
```

## 部署Logstash
```
[root@k8s-master ~]# cd Kubernetes-elk/Logstash/
[root@k8s-master Logstash]# kubectl apply -f configmap.yaml               #创建configmap资源对象
[root@k8s-master Logstash]# kubectl apply -f service.yaml                 #创建service资源对象
[root@k8s-master Logstash]# kubectl apply -f deployment.yaml              #创建deployment资源对象

[root@k8s-master Logstash]# kubectl get deploy,pods,svc,cm -n app | grep logstash
deployment.apps/logstash        1/1     1            1           6d4h
pod/logstash-565f8596b5-vg78r        1/1     Running   0                 11h
service/logstash             ClusterIP   10.100.58.205    <none>        5044/TCP                     6d7h
configmap/logstash-cm        2      6d7h
```

## 安装及配置filebeat
```
[root@nginx ~]# rpm -Uvh https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.14.2-x86_64.rpm
[root@nginx ~]# vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/local/nginx/logs/access.log
  json.keys_under_root: true
  json.overwrite_keys: true
  fields:
    log_app: nginx
    log_topics: nginx_access_logs

output.kafka:
  enabled: true
  hosts: ["kafka-0.kafka-svc:9092"]
  topic: '%{[fields][log_topics]}'
  compression: gzip

[root@nginx ~]# echo "10.110.177.58 kafka-0.kafka-svc" >> /etc/hosts
[root@nginx ~]# systemctl start filebeat
```

## 配置Nginx转发Kibana
```
[root@nginx ~]# vim kibana.conf
server {
    listen 80;
    server_name kibana.qiufeng5.cn;
    location /elk {
        set $service_name kibana-web;
        proxy_http_version 1.1;
        proxy_pass http://10.103.127.77:5601/elk;
    }
}
```

## Kibana可视化展示
![nginx](https://user-images.githubusercontent.com/43721571/145190939-f4b758df-2706-4b16-9beb-25d49fce4205.png)

