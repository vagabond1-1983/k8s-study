# Prometheus的安装和使用
## Prometheus Operator
本文不使用deployment方式部署prometheus，而是用coreos的prometheus operator方式进行部署。因为这种方式提供了大量的默认模板，对prometheus的配置过程简化了非常多，故以下过程是operator的安装和碰到的问题
### prometheus operator安装
- 下载代码
```shell
git clone https://github.com/coreos/kube-prometheus.git
```
- 安装
```shell
cd kube-prometheus/manifests
kubectl create -f setup
kubectl apply -f .
```
- 创建prometheus ingress
```shell
kubectl create -f prome-k8s-ingress.yaml
```
- 创建grafana ingress
```shell
kubectl create -f grafana/grafana-ingress.yaml
```
- 配置域名并访问
## 组件没有采集到数据的问题解决
### kube-scheduler没有采集到数据
kube-scheduler 组件对应的 ServiceMonitor 资源的定义：(prometheus-serviceMonitorKubeScheduler.yaml)没有对应的service，需要创建
prome/prometheus-kubeSchedulerService.yaml
- 创建service
```shell
kubectl create -f prometheus-kubeSchedulerService.yaml
```
- 查看
```shell
kubectl get svc -n kube-system -l k8s-app=kube-scheduler
```
创建完成后，隔一小会儿后去 prometheus 查看 targets 下面 kube-scheduler 的状态
### 修复controller-manager的数据采集
创建一个service
prome/prometheus-kubeControllerMService.yaml
```shell
kubectl create -f prometheus-kubeControllerMService.yaml
```
- 查看
```shell
kubectl get svc -n kube-system -l k8s-app=kube-controller-manager
```
刷新prometheus targets
## 数据持久化
持久化需要用到storageclass对象，需要先创建出来，请移步[storageclass创建过程](../storageclass)

查看prometheus的数据存储
```shell
kubectl get pod prometheus-k8s-0 -n monitoring -o yaml
```
可以看见prometheus-k8s-db的数据挂载卷是emptyDir类型，也就是在随着pod的删除而数据也不会被保存。

由于我们的 Prometheus 最终是通过 Statefulset 控制器进行部署的，所以我们这里需要通过 storageclass 来做数据持久化
- 创建storageclass对象
```shell
kubectl create -f prometheus-storage.yaml
```
- 然后在 prometheus 的 CRD 资源对象中添加如下配置
    ```shell
    vi mainfests/prometheus-prometheus.yaml
    ```
    添加上这段代码
    ```yaml
    torage:
    volumeClaimTemplate:
        spec:
        storageClassName: prometheus-data-db
        resources:
            requests:
            storage: 10Gi
    ```
- 更新下CRD资源
    ```shell
    kubectl apply -f prometheus-prometheus.yaml      
    ```
- 最后查看下是否生效吧
```shell
kubectl get pod prometheus-k8s-0 -n monitoring -o yaml
kubectl get pvc -n monitoring
```    