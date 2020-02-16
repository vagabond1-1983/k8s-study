# storageclass的安装和使用
要使用 StorageClass，我们就得安装对应的自动配置程序，比如我们这里存储后端使用的是 nfs，那么我们就需要使用到一个 nfs-client 的自动配置程序，我们也叫它 Provisioner，这个程序使用我们已经配置好的 nfs 服务器，来自动创建持久卷，也就是自动帮我们创建 PV。
- 自动创建的 PV 以${namespace}-${pvcName}-${pvName}这样的命名格式创建在 NFS 服务器上的共享数据目录中
- 而当这个 PV 被回收后会以archieved-${namespace}-${pvcName}-${pvName}这样的命名格式存在 NFS 服务器上。

可参考github项目 https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client
## [NFS server的安装](nfs-server安装.md)
## nfs client的安装
```shell
kubectl create -f nfsclient-deployment.yaml
kubectl create -f nfsclient-sa.yaml
kubectl create -f nfsclient-sc.yaml
```
## 测试动态PV的创建
```shell
kubectl create -f test-pvc.yaml
```

验证pv
```shell
kubectl get pv
```
自动创建的PV，并绑定了STORAGECLASS managed-nfs-storage

查看host上的存储，多出了一个default开头的目录在/data/k8s下
