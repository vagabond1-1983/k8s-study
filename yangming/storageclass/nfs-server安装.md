# NFS server的安装
- 安装server服务

ubuntu
```shell
apt -y install nfs-kernel-server rpcbind
```
centos
```shell
yum -y install nfs-utils
```
- 设置共享目录在/data/k8s
```shell
chmod 755 /data/k8s/
```
- 配置 nfs
nfs 的默认配置文件在 /etc/exports 文件下，在该文件中添加下面的配置信息：
```shell
vi /etc/exports
/data/k8s  *(rw,sync,no_root_squash)
```
- 启动nfs
```shell
$ systemctl start rpcbind.service
$ systemctl enable rpcbind
$ systemctl status rpcbind

$ systemctl start nfs-server
$ systemctl enable nfs-server
$ systemctl status nfs-server
```