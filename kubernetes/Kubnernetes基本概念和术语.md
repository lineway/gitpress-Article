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

Node节点可以动态的增加到kubernetes集群中，也就是说，如果我们为一台主机安装完成了Node节点所需要的服务并且正常开启，那么这一台节点可以随时添加到正在运行的kuberntes集群中。
Node加入到集群中后，节点上的kubelet进程会定时的将Node节点的信息上报到Master节点中。Master节点根据每个Node节点上报的信息，实现均衡的资源调度策略，一旦某个Node节点在规定时间内没有上报信息，则Master节点判断该Node节点出现异常，然后将该Node节点标记为不可用，然后触发`工作负载迁移`的流程。
在Master节点上，我们可以使用`kubectl get nodes`命令来查看集群中的节点数量及状态，使用`kubectl describe node <node_name>`来查看指定节点的详细信息。

## Pod
Pod是kubernetes集群中一个基本的概念，kubernetes集群能调度的最小单元就是Pod。但是，在一个Pod中可以包含一个或多个容器，其中有一个`pause`的容器比较特殊，这个容器是kuberntes平台的一部分。除了`pause`容器外，每个Pod一般还包含一个或多个用户的业务容器。

在kuberntes中，Pod包含了一组容器单元，这一组容器往往在业务上联系紧密。如果单单使用Docker的话，我们需要编写一个compose文件，将这些容器联系起来，但是，在使用了kuberntes进行编排后，我们就可以将这些容器放在一个Pod中，用户的业务容器可以共享`pause`容器的IP、挂载的Volume等，这样就解决了业务容器之间通信及文件共享的问题。另外，对于在同一个Pod的业务容器而言，我们不需要监控每个容器的运行状态，kubernetes只需要监控`pause`容器的状态，以它的状态代表这个Pod中容器的状态。

在实际的使用中，kubernetes为每个Pod都分配了一个唯一的IP地址，称为PodIP，每个Pod中的容器都共享这个PodIP，通过`虚拟二层网络技术(Flannel、Openvswitch)`，同一kubernetes集群中，Pod之间可以跨越主机进行通信。
### Pod分类
实际上Pod细分下来还可以分为：普通的Pod和静态Pod两种类型。静态Pod比较特殊，它并不存放在集群的etcd存储里，而是放在某个具体的Node上的一个具体文件中，并且只能在这个Node上运行。静态Pod被节点上的kubelet进程直接管理，kubelet通过kubernetes API服务为每个静态pod创建镜像，这些pod对于API服务是可见的，但是不受API服务控制。  
如果确实需要在集群中使用静态pod的话，我们应该考虑DaemonSet的方式。  
Pod可以设置资源限制，用以限制Pod对Node节点上计算资源的使用。在kubernetes中，对于计算资源的限制需要设置两个参数：Requests和Limits，其中Requests表示资源的最小申请量，Limits表示Pod可以使用该资源最大的量，如果Pod申请的资源超过这个限制，那么kubernetes会杀掉该Pod进程并且重启。  

## Label
在Kubernetes中，Label(标签)是附加在资源对象上的一个键-值(key=value)对，key和value都由用户定义。Label与我们日常描述的“标签”大体上相同。给资源对象附加上Label有助于用户进行资源分配、调度、配置、部署等管理工作。  
一个资源对象可以添加任意数量的Label，同样，一个Label也可以附加在任意数量的资源对象上。通过这种方式，可以实现资源的多维度管理工作。给资源对象附加Label可以在资源对象定义时确定，也可以在资源对象创建完成后再动态的添加上去。  
在给资源对象添加完Label后，我们可以使用Label Selector查询及选择具有某些Label的资源对象，然后再进行更多的操作。目前API支持两种selector表达式：基于等式(equality-based)和基于集合(set-based)。  

```python
基于等式的Label Selector
name = mysql:匹配所有具有Label为name=mysql的资源对象
environment != development:匹配所有Label中不具有environment=development的资源对象

基于集合的Label Selector
name in ()

```