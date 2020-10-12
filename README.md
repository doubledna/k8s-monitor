## 用于部署kubernetes集群监控

## 使用到镜像都上次到对应环境的harbor仓库中

## 注意事项：当有新的namespace需要监控，需要在prometheus-rbac.yaml中添加对应的Role和RoleBinding,相对应巴西k8s集群有prod-phi namespace 需要在indo集群也创建这个namespace，防止安装prometheus-rbac报错

## 部署步骤

* 1、部署该服务的第一步是创建一个叫monitoring的namespace
- kubectl create namespace monitoring

* 2、部署prometheus-operator
- kubectl create -f bundle.yaml
- kubectl create -f prometheus-operator-servicemonitor

* 3、部署 prometheus

* 4、部署 kube-state-metrics, （问题：无法获取到deployment问题，解决方法是在kube-state-metrics的rbac中的Role的resource添加 deployment资源对象

* 5、部署 metrics-server （碰到的问题，测试环境metric-server部署失败，原因：1、未开启API aggregator功能（需要生产api访问metrics的证书） 2、master节点需要安装flannel服务)

* 6、部署kube-server ,这里因为二进制安装k8s集群，controller、scheduler的endpoint需要自己去处理，先安装对应地区的endpoints文件，再安装servicemonitor

* (碰到的问题，无法收集到controller、scheduler的数据，发现是kube-controller-manager.service,kube-scheduler.service配置文件中配置的端口是127.0.0.1,导致无法访问到服务需要修改成0.0.0.0,修改完成后发现 controller还是有问题请求是http连接，所以需要将kube-controller-manager.service中 --port=0 --secure-port=10252 注释掉)

## 各个安装组件作用

* prometheus-operator: prometheus-operator的本职就是一组用户自定义的CRD资源以及Controller的实现，Prometheus Operator这个controller有BRAC权限下去负责监听这些自定义资源的变化，并且根据这些资源的定义自动化的完成如Prometheus Server自身以及配置的自动化管理工作

* prometheus: 用于监控数据的聚合等

* kube-state-metrics: 用于获取如pod，service,deployment等资源对象的状态信息

* metrics-server: 用于获取node，pods的CPU，Memory信息

* kube-server: 用于获取k8s各个组件的信息

* alertmanager: 用于prometheus报警插件：

* 在安装alertmanager需要将自己的alertmanager.yml映射到alertmanager pod中 操作如下：

    * 1、删除原来的alertmanager-main secret          
    -  kubectl delete secret alertmanager-main -n monitoring   

    * 2、到alertmanager_yaml/对应地区/目录下，将新的alertmanager.yaml create成新的secret  
    - kubectl create secret generic alertmanager-main --from-file=alertmanager.yaml -n monitoring

