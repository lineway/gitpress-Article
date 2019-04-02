# Kubernetes基本概念和术语

kubernetes中的大部分概念（Node、Replication Controller、Service）等都可以看作是资源对象，这些资源对象都可以通过API编程调度（kubectl工具就是封装了这些调度接口），在调度完成后，将资源的状态保存在`etcd`中持久化存储。
## Master
在kubernetes集群中，Master指的就是集群的控制节点，这个节点主要负责集群的管理和控制。在实际的使用中，我们对于集群的操作都是发送到Master节点，由它负责具体的执行过程。其中Master节点运行着以下相关进程：
 - Kubernetes API Server(kube-apiserver):该进程提供HTTP Rest的接口，集群中对资源的操作都是通过它来实现的，是整个集群的入口；
 - Kubernetes Controller Manager(kube-controller-manager):集群中