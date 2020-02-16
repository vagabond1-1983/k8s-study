# Ingress的安装过程
ingress是访问k8s集群内部网络的解决方案，有很多解决方案，本文使用了traefik的解决方案。
## 安装traefik
### 安装HTTPS服务的准备工作
- openssl命令生成CA证书
```shell
openssl req -newkey rsa:2048 -nodes -keyout tls.key -x509 -days 365 -out tls.crt
```
- kubectl创建secret对象
```shell
kubectl create secret generic traefik-cert --from-file=tls.crt --from-file=tls.key -n kube-system
```
- 查看secret对象
```shell
kubectl get secret traefik-cert -n kube-system -o yaml
```
### 创建ConfigMap对象，内容为traefik的配置文件，配置https服务端口
```shell
kubectl create configmap traefik-conf --from-file=traefik.toml -n kube-system
```
验证是否创建成功
```shell
kubectl get cm traefik-conf -n kube-system
```
### 创建一个rbac权限访问
```shell
kubectl create -f rbac.yaml
```
### 创建traefik
```shell
kubectl create -f traefik.yaml
```
### 创建ingress，通过域名访问traefik管理UI
```shell
kubectl create -f ingress.yaml
```
