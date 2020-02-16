# 安装单节点k8s集群
网上有很多使用kubeadm安装k8s单集群的例子，我这里不做赘述，列出主要的步骤。
建议用[这篇文章](https://www.qikqiak.com/k8s-book/docs/16.%E7%94%A8%20kubeadm%20%E6%90%AD%E5%BB%BA%E9%9B%86%E7%BE%A4%E7%8E%AF%E5%A2%83.html)做参考。

下面是我的搭建之路，有点乱，在此记录下我的遇坑过程。

=================================

以下操作在ubuntu 18.04环境下进行
1. 安装docker环境
- 安装docker
```shell
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
sudo systemctl enable docker
sudo systemctl start docker
```
- 将当前用户加入docker组
```shell
sudo usermod -aG docker $USER
```

- 镜像加速

对于使用systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）
```shell
sudo mkdir -p /etc/docker 
sudo tee /etc/docker/daemon.json <<-'EOF' { "registry-mirrors": ["https://6yym4nbo.mirror.aliyuncs.com"] } EOF
```
- 之后重新启动服务
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

- 禁用swap

编辑/etc/fstab文件，注释掉引用swap的行
```shell
sudo swapoff -a
```
- 解除防火墙限制
```shell
vi /etc/sysctl.conf
```
最后一行添加
```sh
net.bridge.bridge-nf-call-ip6tables = 1
```
接着输入命令，使ipables更改生效
```shell
sysctl -p
```


1. 安装kubeadm
对于不用翻墙的，请忽略下面的2.1节
2.1 更新软件源
- 安装GPG证书
```shell
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
```
- 更新软件源
```shell
cat << EOF >/etc/apt/sources.list.d/kubernetes.list  
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main  
EOF
```
2.2 安装kubeadm,kubelet和kubectl
- 更新
```shell
apt-get update && apt-get install -y apt-transport-https curl
```
- 安装软件
```shell
apt-get install -y kubelet kubeadm kubectl
```
- 设置kubelet自启动，并启动kubelet
```shell
systemctl enable kubelet && systemctl start kubelet 
```
- 配置CGROUP
查看 Docker 使用 cgroup driver:
```shell
docker info | grep -i cgroup
-> Cgroup Driver: cgroupfs
```
而 kubelet 使用的 cgroupfs 为system，不一致故有如下修正：
```shell
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
加上如下配置：
```sh
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```
或者
```sh
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
```
- 重启kubelet
```shell
systemctl daemon-reload
systemctl restart kubelet
```
3. 初始化kubeadm
```shell
kubeadm init --pod-network-cidr=192.168.0.0/16
```
4. 验证
输入命令：
```shell
kubectl get nodes
```
得到的结果是8080 connection failed。说明api-server的端口没有被识别
为了使用kubectl访问apiserver，在~/.bashrc中追加下面的环境变量：
```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```
然后更改生效
```shell
source ~/.bashrc
```
这时再输入命令
```shell
kubectl get nodes
NAME              STATUS     ROLES    AGE   VERSION
vaga-virtualbox   NotReady   master   16m   v1.15.3
```
此时的集群状态是NotReady的，需要安装网络插件
5. 安装flannel网络
```shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
```
**需要注意的是如果你的节点有多个网卡的话，需要在 kube-flannel.yml 中使用--iface参数指定集群主机内网网卡的名称，否则可能会出现 dns 无法解析。flanneld 启动参数加上--iface=<iface-name>**
```sh
args:
- --ip-masq
- --kube-subnet-mgr
- --iface=eth0
```
6. 验证
这个时候集群应该就是Ready状态了。
