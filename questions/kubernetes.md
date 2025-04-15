#### 1.控制面组件和作用

控制面组件包括`kube-apiserver`，`etcd`，`kube-controller-manager`，`kube-scheduler`和`cloud-controller- manager`

`kube-apiserver`是外部与Kubernetes集群交互的唯一入口，核心功能是`认证`、`授权`和`准入控制`，是Kubernetes集群的消息总线，负责与`etcd`交互，将obj对象写入`etcd`。

`etcd`是一个分布式kv数据库，负责持久化存储整个集群的配置和状态信息，采用raft一致性算法保证数据一致性和高可用性，支持Watch API与kube-apiserver建立长链接，间接为其他组件提供数据。

`kube-controller-manager`负责维护集群的期望状态，其中包含多个二进制组件，每个控制器组件管理一种资源对象，如果对象的实际状态和期望状态不一致，则通过调谐使其达到期望状态。

`kube-scheduler`负责为Pod选出最优节点，通过在插件化的framework中注册各种plugin，并在对应的阶段执行插件对目标节点做筛选，调度的过程中包括调度周期和异步的绑定周期。

`cloud-controller- manager`使负责与云服务提供商进行交互的组件，它使得Kubernetes集群能够更好地与云环境集成，充分利用云平台提供的各种资源和服务。

#### 2.如果从零开始部署集群



#### 3.什么是Pod



#### 4.paues容器和其生命周期



#### 5.探针有哪几种



#### 6.Pod创建过程



#### 7.集群中的事件是如何产生的



#### 8.Pod处于Pending状态如何排查



#### 9.预选个优选的区别



#### 10.容器运行时切换步骤



#### 11.Service如何管理Endpoint



#### 12.边车对服务有什么影响



#### 13.通过Ingress暴露的服务不可用如何排查



#### 14.对静态容器的理解



#### 15.ClusterIP是如何实现的



#### 16.Operator如何开发



#### 17.二进制部署和kubeadm部署集群的对比



#### 18.如何实现集群的高可用



#### 19.对比各种CNI插件



#### 20.CRI都提供哪些服务，如何工作