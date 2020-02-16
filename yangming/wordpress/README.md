# 安装wordpress应用
通过创建wordpress应用的过程，练习deployment类型应用的创建过程，及如何升级为域名访问和HPA的过程。
## 创建mysql数据库
- 创建新的namespace
```shell
kubectl create namespace blog
```
- 创建mysql数据库
```shell
kubectl create -f wordpress-db.yaml
```
- 查看db的service
```shell
kubectl get svc -n blog
NAME    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
mysql   ClusterIP   10.99.9.149   <none>        3306/TCP   2m29s

kubectl describe svc mysql -n blog
Name:              mysql
Namespace:         blog
Labels:            <none>
Annotations:       <none>
Selector:          app=mysql
Type:              ClusterIP
IP:                10.99.9.149
Port:              mysqlport  3306/TCP
TargetPort:        dbport/TCP
Endpoints:         10.244.0.15:3306
Session Affinity:  None
Events:            <none>
```
## 创建wordpress应用
- 创建wordpress应用
```shell
kubectl create -f wordpress.yaml
deployment.apps/wordpress-deploy created

kubectl get pods -n blog
NAME                               READY   STATUS    RESTARTS   AGE
mysql-deploy-966f474bb-psc9g       1/1     Running   0          7m30s
wordpress-deploy-7f689b5db-2w8zs   1/1     Running   0          3m8s
```
- 添加service访问wordpress应用
可以添加一个NodePort的service进行访问，这里暂时不用此种方式进行访问。后面用ingress的方式域名访问。
## 创建高可用的wordpress应用
```shell
kubectl create -f wordpress-hpa.yaml
```
这样就创建了一个高可用的wordpress应用。
参见wordpress-hpa.yaml文件，主要添加了以下几点：
- 可读探针，监听80端口，启动后5秒查看80端口情况，每隔10秒探查一次
```yaml
readinessProbe:
  tcpSocket:
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10 
```
- 同一个pod中通过名称访问容器
此处mysql就是指向的mysql容器，而不使用集群的ip
```yaml
env:
  - name: WORDPRESS_DB_HOST
    value: mysql:3306
```
- 添加initContainer做初始化动作
wordpress需要在mysql可用后，利用nslookup查询mysql服务是否注册
```yaml
initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nslookup mysql; do echo waiting for mysql service; sleep 2; done;']
```
- 添加资源限制
限定wordpress这个deployment的机器资源
```yaml
resources:
  limits:
    cpu: 200m
    memory: 200Mi
  requests:
    cpu: 100m
    memory: 100Mi
```   
- 设置滚动更新策略，只能运行1个并必须有1个稳定运行
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1 
- HPA，当流量变大可以自动扩容，变小可以自动缩容
```shell
kubectl autoscale deployment wordpress-deploy --cpu-percent=10 --min=1 --max=10 -n blog
```    
### 创建一个ingress对象，域名访问wordpress应用
```shell
kubectl create -f wordpress-svc-ingresss.yaml
```
这样就可以通过域名方式访问wordpress应用了。
### 测试高可用，应用的伸缩
```shell
source test-hpa.sh
```
这个脚本会一直访问wordpress应用
查看hpa资源
```shell
kubectl get hpa -n blog
```