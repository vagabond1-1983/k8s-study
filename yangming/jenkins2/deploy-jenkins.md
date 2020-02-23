# 部署k8s架构下的Jenkins
## Jenkins master和slave都是k8s的pod
此种方式就是将jenkins的master和slave都部署到k8s集群中，利用k8s架构管理jenkins。
### 部署master
- 创建kube-ops命名空间
```shell
kubectl create namespace kube-ops
```
- jenkins master rbac，添加名叫jenkins2的sa的集群访问权限。
此权限限制在kube-ops命名空间下，后面需要在jenkins的管理端设置  
```shell
kubectl create -f jenkins-rbac.yaml
```
- jenkins master pvc，用于jenkins home的文件存储，连接的是之前的nfs server
```shell
kubectl create -f jenkins-pvc.yaml
```
- jenkins master deployment，也是创建在kube-ops命名空间下
```shell
kubectl create -f jenkins.yaml
```
### 查看jenkins pod
```shell
kubectl logs -f jenkins2-7bb76bbf7f-m8qv4 -n kube-ops
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```
很明显可以看到上面的错误信息，意思就是我们没有权限在 jenkins 的 home 目录下面创建文件，这是因为默认的镜像使用的是 jenkins 这个用户，而我们通过 PVC 挂载到 nfs 服务器的共享数据目录下面却是 root 用户的，所以没有权限访问该目录，要解决该问题，也很简单，我只需要在 nfs 共享数据目录下面把我们的目录权限重新分配下即可：
```shell
chown -R 1000 /data/k8s/jenkins2
```
然后重建
```shell
kubectl delete -f jenkins.yaml
deployment.apps "jenkins2" deleted
service "jenkins2" deleted
kubectl create -f jenkins.yaml
deployment.apps/jenkins2 created
service/jenkins2 created
```

再次查看发现jenkins pod running了。
```shell
kubectl get pods -n kube-ops
NAME                        READY   STATUS    RESTARTS   AGE
jenkins2-7bb76bbf7f-9sjhr   1/1     Running   0          23s
```
### 配置kubernetes
要想jenkins创建动态的slave，即slave是以pod形式运行的，需要安装和配置kubernetes
- 首先下载插件kubernetes和blue ocean
- 用admin账户登陆后，进入系统管理 -> 系统配置 -> 云
kubenates地址填写：https://kubernetes.default.svc.cluster.local
kubenates命令空间填写：kube-ops
然后点击测试，如果测试通过，就说明jenkins master和kubernetes api server建立了连接。

## 在已有的jenkins master下连接kubernetes集群
这种方式是已经有jenkins master节点了，想要连接kubernetes集群，这时就不能创建jenkins deployment了。只需要在
系统管理 -> 系统配置 -> 云配置好就可以了。
- 首先下载插件kubernetes和blue ocean
- 用admin账户登陆后，进入系统管理 -> 系统配置 -> 云
- kubenates地址填写
找到k8s下的kubernetes这个svc，并编辑为NodePort
```shell
kubectl edit svc kubernetes
type: NodePort
```
查看这个service的NodePort端口
```shell
kubectl get svc
```
然后写入到kubenates地址
- kubenates命令空间填写：kube-ops
- 凭据
这个凭据非常重要，需要把你账户的config文件生成一个证书，然后上传到凭据中。
首先是config文件应该存放在你的.kube/config中，里面有三段加密串，certificate-authority-data, client-certificate-data和client-key-data 分别生成文件ca.crt，client.crt，client.key。注意生成文件时需要用base64 -D进行解码，下面举个例子：
```shell
echo "xxxx" | base64 -D
----START CERTIFICATION----
....
----END CERTIFICATION----
```
然后用下面的命令生成一个证书文件cert.pfx 
```shell
openssl pkcs12 -export -out cert.pfx -inkey client.key -in client.crt -certfile ca.crt
```
在凭据中添加一个证书类型的凭据，并上传cert.pfx。
- 测试
点击测试，通过后说明连接建立成功啦。

## [jenkins安装的插件汇总](jenkins-plugins.md)