# Jenkins下的动态slave实现
## [回看部署k8s架构下的Jenkins](deploy-jenkins.md)
## 通用模板配置动态slave
1. 在系统配置-kubernetes-pod模板，添加一个通用的模板
命名空间：kube-ops
标签列表：xxx
在容器列表中添加容器模板
Container Template:
    Docker镜像：cnych/jenkins:jnlp6
    注意：运行的命令和命令参数里的内容删除
卷：
添加一个docker的Host Path Volume和kube的Host Path Volume，前者为了使用docker命令，后者为了使用kubectl命令
主机路径：/var/run/docker.sock
挂载路径：/var/run/docker.sock  

主机路径：/root/.kube
挂载路径：/root/.kube
Service Account:
注意这里需要填写上对kube-ops有权限的rbac，jenkins2。因为这个sa是之前安装环境的时候创建的。
2. 测试
新建一个流水线任务，在pipeline script写上：
```groovy
node('kong-jnlp') {
    stage('docker info') {
        sh "docker info"
    }

    stage('kubectl') {
        sh "kubectl get no"
    }
}
```
3. 观察
观察下kube-ops是否有名称为slave-开头的pod生成。
从job的blue ocean视图，可以看到流水线的过程。
## Jenkinsfile配置动态slave
1. 去除通用的pod template，这种方式不适合pod模板不固定的jenkins管理，需要在jenkinsfile中通过代码形式写pod template
2. [编写Jenkinsfile](aslave/Jenkinsfile)
3. 新建流水线任务，在pipeline节点选择git，并配置HTTPS形式的git仓库地址。
4. 构建完检查blue ocean和kube-ops下的pod生成情况。
