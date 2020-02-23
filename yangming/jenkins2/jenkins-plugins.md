# jenkins需要安装的插件
- git
- gitlab/github
- groovy
- pipeline -- 流水线job必须
- blue ocean -- 流水线视图必须
- kubernetes -- k8s集群必须
- prometheus metrics -- 通过暴露metrics给prometheus，监控jenkins运行状态和资源情况
- sonarqube scanner -- sonar代码扫描
- pipeline maven -- 构建maven项目，可自定义settings等配置