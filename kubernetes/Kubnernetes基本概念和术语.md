# Kubernetes基本概念和术语

kubernetes中的大部分概念（Node、Replication Controller、Service）等都可以看作是资源对象，这些资源对象都可以通过API编程调度（kubectl工具就是封装了这些调度接口），在调度完成后，将资源的状态保存在`etcd`中持久化存储。
## Master
在kubernetes集群中，Master指的就是集群的控制节点，这个节点主要负责集群的管理和控制。在实际的使用中，我们对于集群的操作都是发送到Master节点，由它负责具体的执行过程。其中Master节点运行着以下相关进程：
 - Kubernetes API Server(kube-apiserver):该进程提供HTTP Rest的接口，集群中对资源的操作都是通过它来实现的，是整个集群的入口；
 - Kubernetes Controller Manager(kube-controller-manager):集群中资源对象的控制中心；
 - Kubernetes Scheduler(kube-scheduler):集群中负责资源调度的进程；
 - etcd Server:该进程并不是必选项，一般而言我们默认使用etcd服务来保存资源对象的数据。

## Node
Kubernetes集群中，其他节点统称为Node节点。Master和Node可以是物理主机也可以是虚拟主机。在Kubernetes集群中，Node节点是负责工作负载 的主体。在Node节点上，一般会运行着以下进程：
 - kubelet:负责Pod对应的容器的创建、启动、停止等工作，与Master协作，实现集群管理的功能；
 - kube-proxy:该进程主要负责通信及负载均衡机制；
 - Container Engine:容器引擎，这里可以选择Docker或者kubernetes支持的容器引擎，一般我们选择Docker Engine。

Node节点可以动态的增加到kubernetes集群中，也就是说，如果我们为一台

