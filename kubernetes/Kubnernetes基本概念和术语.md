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
name in (redis-master, redis-salve):匹配所有Label包含name=redis-master和name=redis-slave的资源对象
tier notin (frontend, backend):匹配所有Label中不包含tier=frontend和tier=backend的资源对象
```
Label Selector还可以通过组合实现更复杂的查询，每个查询条件使用“,”进行分割，代表的是逻辑与(&&)的关系。比如：
```python
name=mysql,env!=development:代表Label中包含name=mysql，不包含env=development的资源对象
```
Label Selector的作用主要体现在以下几个方面：
 - kube-controller进程通过资源对象RC上定义的Label Selector来筛选要监控的pod副本数量；
 - kube-proxy进程通过Service的Label Selector来选择对应的pod，自动建立起每个Service到对应pod的请求转发路由；
 - 通过对某些node定义Label，通过kube-scheduler定向的将pod调度到某些node上。
通过使用Label和Label Selector，可以更方便的实现对资源对象的精细化管理，以及实现集群的高可用性。

## Replication Controller(RC)

对于Replication Controller的官方解释是，Replication Controller确保一次运行指定数量的pod，确保一个pod或一组同类的pod总是可用的。我们可以理解为，Replication Controller对pod的运行进行了定义，一旦pod不符合这个定义，就进行相应的调整。

那么我们在定义一个Replication Controller的时候，需要定义哪些内容呢？一般而言，我们需要包含以下信息：

- Pod的副本数量（replicas）；
- 用于选择目标pod的Label Selector；
- 创建新pod使用的模板（template）。

以下是一个Replication Controller的定义文件：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
    name: nginx
spec:
    replicas: 3
    selector:
        app: nginx
    template:
        metadata:
            name: nginx
            labels:
                app: nginx
... ...
```

上述文件中，`kind`声明了该配置文件是一个`ReplicationController`，文件中包含了之前提到的内容，`replicas`定义了pod的数量，`selector`定义了Label Selector，`template`中定义了创建新pod使用的模板。

当定义的RC提交到kubernetes集群以后，Master节点上的Controller Manager会定期的在集群中巡检pod，如果发现pod数量与RC中定义的数量不符合时，就会删除多余的pod或者创建新的pod以保证集群中pod的数量与RC中定义的数量一致。这就实现了集群的高可用性。**如果可能的话，即使创建单个pod，我们也尽可能的使用Replication Controller的方式**。当然，在运行时，我们也可以通过命令修改RC的replicas参数，动态的调整pod的数量来实现缩放（scaling），这可以通过`kubectl scale`命令来实现，如：

```bash
kubectl scale rc nginx --replicas=5
```

使用上述命令对pod进行缩放后，可以在集群中查看pod数量是否与修改后的数量一致。

对于RC中定义的pod，在删除的时候，首先将RC中的`replicas`数量设置为0，然后更新该RC，也可以通过`kubectl`提供的`stop`和`delete`命令来删除RC和RC中定义的pod。

### 滚动升级

由于RC的存在，当我们在切换集群中某类pod中的Docker镜像的时候，不是一次性完成镜像的更换，而是逐步更换镜像，保持集群中总的pod数量是不变的，当所有的pod中的镜像完成更换后，升级完成。这种方式称为滚动升级（Rolling Update）。

### Replica Set

在新版本的kubernetes中，Replication Controller升级为一个新的概念----ReplicaSet，相比于Replication Controller而言，ReplicateSet增加了基于集合的Label Selector（Set-based selector），RC只支持基于等式的Label Selector（equality-based selector）。

我们来看一个官方的ReplicaSet的例子：

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3

```

其中`matchExpressions`下的条件`{key: tier, operator: In, values: [frontend]}`就是定义的匹配条件及`matchLabel`用于查找详细信息的操作。

对于RS(Replica Set)来说，`kubectl`中可用于RC的命令大部分都适用，比如我们可以使用`kubectl get rs`来查看RS的信息。

但是，实际上我们很少会单独使用Replica Set，其主要是被Deployment这个资源对象使用，Deployment管理Replica Set并为pod提供更新及其他功能。也就是说，除非需要自定义编排，我们没有必要直接操作Replica Set，而是直接使用Deployment。

## Deployment

实际上，Deployment是一个高级的、升级版的Replication Controller。相对于RC来说，Deployment可以方便的直到当前pod的部署进度。

Deployment的典型使用场景有以下几个方面：

- 创建一个Deployment对象来生成对应的Replica Set并完成pod副本的创建过程；
- 更新Deployment以创建新的Pod（如镜像升级）；
- 回滚到之前的Deployment版本；
- 扩展Deployment以应对更多负载；
- 清除不在需要的Replica Sets；
- 挂起或恢复一个Deployment；
- 检查Deployment的状态以确定部署是否完成；

Deployment的定义与Replica Set的定义很相似，区别仅限于API声明及kind类型：

```yaml
apiVersion: apps/v1beta1 # for versions before 1.6.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

完成一个Deployment的编写后，使用`kubectl create -f <deployment-file>.yaml`来创建一个Deploymnet，使用`kubectl get deployments`命令来查看Deployment的信息。，如果想查看更信息的扩展信息，可以使用`kubectl describe deployments`来查看输出结果。

## Service

kubernetes的Service定义了一种抽象对象，即逻辑上的一组Pod和一种可以访问它们的策略——微服务。这一组Pod能够被Service访问到，通常是通过Label Selector实现的。

对于Serivce来说，对接的一组Pod中，每个Pod在kubernetes中都会被分配一个独立的IP地址，并且每个Pod都有单独的Endpoint对外提供服务。因此，Service对外提供服务的时候，是需要在这组Pod组成的集群中添加一个负载均衡器，将这一组的Pod的Endpoint列表加入到负载均衡器的端口转发列表中。这样以来，外部客户端通过访问负载均衡器的对外IP地址和端口就可以访问Service，最后外部客户端的请求具体转发到哪一个Pod中则是由负载均衡器的算法分配的。  
在kubernetes中，运行在每个Node上的kube-proxy就是一个负载均衡器，它主要负责将对Service的请求转发到后端的Pod实例上，并在内部实现服务的负载均衡和会话保持机制。需要注意的是，Service不是共用一个负载均衡器的IP地址，而是每个Service分配了全局唯一的虚拟IP地址，称为ClusterIP。这样一来，每个Service就变成了具备唯一IP地址的端点，服务调用就变成了基本的TCP通信问题。  
在前面的内容中，我们知道Pod的Endpoint会随着Pod的销毁和重新创建而发生改变。但是，Service一旦创建，kubernetes就自动的分配一个可用的ClusterIP，在整个Service的生命周期内，它的ClusterIP不发生变化，那么我们只要将Service的Name与Service的ClusterIP做一个DNS域名映射，服务发现的问题也就相应的被解决了。  
在实际应用中，很多服务其实是有多端口的，比如：MySQL、redis等。Service支持多个Endpoint，如果存在多个Endpoint的时候，分别对Endpoint命名进行区分，对于多端口命名的操作，就牵扯服务发现这部分的内容，后续会做详细介绍。  
### 外部系统访问Service的问题
在目前介绍的概念中，我们了解到kubernetes中已经存在三种不同的IP，分别是`Node IP`、`Pod IP`和`Cluster IP`。它们的作用分别是：
 - `Node IP`是集群中每个节点的物理网卡IP地址，这是一个真是存在的物理网络，所有在这个网络中的服务器都可以通过这个网络进行通信，不管它是不是在集群中。这也说明集群之外的节点访问集群内的某个节点或者TCP/IP服务的时候，必须通过Node IP进行通信；
  - `Pod IP`是每个Pod的IP地址，这是通过docker0网桥的IP地址段进行分配的，通常是一个虚拟的二层网络。因此在集群中，不同的Pod进行通信的时候，就是通过Pod IP所在的虚拟二层网络进行通信的，真实的TCP/IP流量则是通过Node IP所在的物理网卡流出的；
  - `Cluster IP`是一个虚拟的IP，它仅仅作用于Service对象，并有kubernetes管理和分配IP地址（来源于Cluster IP池），使用ping命令不能连接到Cluster IP，因为没有网络对象进行响应，最后，Cluster IP只能结合Service Port组成一个具体的通信端口，单独的Cluster IP不具备TCP/IP通信的基础。

从上面可以看出，Cluster IP属于kubernetes内部的地址，无法在集群外部直接使用这个地址。为了能够让外部访问Service，可以采用NodePort的方式。具体可以在Service的配置中做如下定义：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  type: NodePort
  ports:
   - ports: 8080
     nodePort: 30001
  selector:
    tier: frontend
```
上述配置信息中，`spec.type`表明指定tomcat-service使用NodePort，`spec.ports.nodePort`定义了其使用的端口为30001，这样以来，在浏览器中通过`http://<nodePort>:30001`就可以访问tomcat了。  
NodePort的实现实质上是kubernetes集群为Node上需要外部访问的Service开启一个对应的TCP监听端口。  
#### 负载均衡
前面介绍的内容解决了集群外部访问Service的主要问题，但是还有负载均衡问题没有得到处理。如下所示：
![loadbalancer.png](https://i.loli.net/2019/04/09/5cac6390a4160.png)
我们设置负载均衡独立于kubernetes集群之外，外部请求的流量会首先经过负载均衡器，然后由负载均衡器分配带集群的Node上，实现这个功能可以通过加设硬件负载均衡器或通过HAproxy或nginx的方式来配置流量转发。  