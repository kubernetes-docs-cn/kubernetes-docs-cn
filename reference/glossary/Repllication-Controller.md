### Replication Controller

#####什么是Replication Controller
Replication Controller简称副本控制器，它能保证在任何时候，都运行用户期望数量的pod。如果pod过多，则会杀死多余的pod，如果pod过少，则会自动增加pod，以达到用户期望的数量的pod。与手动创建的pod不同，如果复制控制器维护的pod失败，被删除或终止，它们将被自动替换。例如，在中断性维护（例如内核升级）之后，您的pod会在节点上重新创建。因此，我们建议您使用复制控制器，即使您的应用程序只需要一个单元。您可以将复制控制器看作类似于流程管理程序的东西，而不是单个节点上的单个进程，复制控制器监控多个节点上的多个pod。
复制控制器在讨论中通常缩写为“rc”或“rcs”，并且作为kubectl命令中的快捷方式。
一个简单的情况是创建1个Replication Controller对象，以便可靠地无限期地运行Pod的一个实例。更复杂的用例是运行复制服务的多个相同副本，例如Web服务器等

#####运行一个rc的demo
下面是一个示例Replication Controller配置。 它运行3副本的nginx Web服务器。
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
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
通过如下命令创建rc
```
 kubectl create -f ./replication.yaml
```
查看rc的详细情况
```
$ kubectl describe replicationcontrollers/nginx
Name:		nginx
Namespace:	default
Image(s):	nginx
Selector:	app=nginx
Labels:		app=nginx
Replicas:	3 current / 3 desired
Pods Status:	0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Events:
  FirstSeen				LastSeen			Count	From
SubobjectPath	Reason			Message
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-qrm3m
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-3ntk0
  Thu, 24 Sep 2015 10:38:20 -0700	Thu, 24 Sep 2015 10:38:20 -0700	1
{replication-controller }			SuccessfulCreate	Created pod: nginx-4ok8v
```
这里，已经做出了3个pod，但是还没有运行，可能是因为图像被拉。 稍后，同样的命令可能会显示：
```
Pods Status:	3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```
#####复制控制器规范
所有其他Kubernetes配置一样，rc需要apiVersion，kind和metadata字段。有关使用配置文件的一般信息，请参见[此处](http://kubernetes.io/docs/user-guide/simple-yaml/ "此处")，[此处](http://kubernetes.io/docs/user-guide/configuring-containers/ "此处")和[此处](http://kubernetes.io/docs/user-guide/working-with-resources/ "此处")。
复制控制器还需要一个.spec部分

#####pod模版
.spec.template是.spec的唯一必填字段。
.spec.template是一个pod模板。它与pod有完全相同的模式，除了它是嵌套的，没有apiVersion或kind。
除了Pod的必填字段外，Replication Controller中的pod模板必须指定适当的标签（即不与其他控制器重叠，请参阅pod选择器）和适当的重新启动策略。
只允许等于Always的.spec.template.spec.restartPolicy，如果未指定，则为默认值。
对于本地容器重新启动，复制控制器委派给节点上的代理，例如Kubelet或Docker。

#####rc上的标签

复制控制器本身可以有标签（.metadata.labels）。通常，您将设置与.spec.template.metadata.labels相同;如果未指定.metadata.labels，则默认为.spec.template.metadata.labels。但是，允许它们不同，.metadata.labels不会影响复制控制器的行为。

#####pod选择器
.spec.selector字段是一个标签选择器。复制控制器管理具有与选择器匹配的标签的所有pod。它不会将其创建或删除的pod与其他人或进程创建或删除的pod区分开。这允许在不影响正在运行的pod的情况下替换复制控制器。
如果指定，.spec.template.metadata.labels必须等于.spec.selector，否则它将被API拒绝。如果.spec.selector未指定，它将默认为.spec.template.metadata.labels。
此外，您通常不应创建标签匹配此选择器的任何pod，直接，通过另一个ReplicationController或通过另一个控制器（如Job）。否则，ReplicationController会认为这些pod是由它创建的。 Kubernetes不会阻止你这样做。
如果你最终有多个控制器有重叠的选择器，你将不得不自己管理删除（见下文）。
#####多个副本

您可以通过将.spec.replicas设置为您希望同时运行的pod数量来指定应同时运行多少个pod。在任何时间运行的数字可以更高或更低，例如，如果副本刚刚增加或减少，或者如果pod正常关闭，并且替换开始提前。
如果不指定.spec.replicas，那么它默认为1

#####使用rc

######删除rc及其Pods

要删除复制控制器及其所有pod，请使用kubectl delete。 Kubectl将缩放复制控制器为零，并等待它删除每个pod，然后再删除复制控制器本身。如果此kubectl命令中断，则可以重新启动。
使用REST API或go客户端库时，需要明确执行这些步骤（将replica复制到0，等待pod删除，然后删除复制控制器）。
######仅删除rc

您可以删除复制控制器，而不影响其任何pod。
使用kubectl，对kubectl delete指定--cascade = false选项。
使用REST API或转到客户端库时，只需删除复制控制器对象。
删除原件后，可以创建新的复制控制器来替换它。只要旧的和新的.spec.selector是相同的，那么新的将采用旧的pods。但是，它不会做任何努力，使现有的pods匹配一个新的，不同的pod模板。要以受控的方式将pod更新为新规范，请使用滚动更新。
######从rc隔离pod

可以通过更改其标签从复制控制器的目标集中删除Pod。这种技术可以用于从服务中移除pod以用于调试，数据恢复等。以这种方式移除的pod将被自动替换（假设副本的数量也不改变）

#####常见的使用模式

######重新调度

如上所述，无论您是要保持运行的1个pod还是1000个，复制控制器都将确保指定数量的pod存在，即使在节点故障或pod终止的情况下（例如，由于另一个控制剂）。
######水平扩展

复制控制器通过简单地更新副本字段，可以轻松地手动或通过自动缩放控制代理向上或向下缩放副本的数量。
######滚动升级
复制控制器旨在通过逐个替换pod来帮助滚动更新服务。
建议的方法是创建具有1个副本的新复制控制器，逐个扩展新（+1）和旧（-1）控制器，然后在旧控制器达到0个副本后删除旧控制器。这可预测地更新pod的集合，而不管意外的故障。
理想地，滚动更新控制器将考虑应用程序就绪，并且将确保在任何给定时间有效地服务足够数量的pod。
两个复制控制器将需要创建具有至少一个区分标签的pod，例如pod的主容器的图像标签，因为它通常是激励滚动更新的图像更新。
滚动更新在客户端工具kubectl滚动更新中实现。访问kubectl滚动更新教程了更具体的例子。
#####将rc与servicec配合使用

多个复制控制器可以位于单个服务之后，例如，一些流量转到旧版本，一些转到新版本。
复制控制器永远不会自己终止，但不会像服务一样长时间运行。服务可以由由多个复制控制器控制的pod组成，并且期望在服务的生命周期内创建和销毁许多复制控制器（例如，执行运行服务的pod的更新）。服务本身和他们的客户端应该仍然没有看到维护服务的pod的复制控制器。
#####编写程序以进行复制

由复制控制器创建的Pod是意图可互换和语义上相同的，尽管它们的配置可能随着时间变得异构。这显然适合复制的无状态服务器，但复制控制器也可用于维护主控选择，分片和工作程序池应用程序的可用性。这样的应用程序应该使用动态工作分配机制，例如etcd锁定模块或RabbitMQ工作队列，而不是静态/一次性定制每个pod的配置，这被认为是反模式。执行的任何pod定制，例如资源（例如，cpu或内存）的垂直自动调整，应该由另一个在线控制器进程执行，这与复制控制器本身不同

#####rc的职责

复制控制器简单地确保所需数量的盒匹配其标签选择器并且可操作。目前，只有已终止的pod被排除在其计数之外。在未来，可以考虑从系统可用的准备和其他信息，我们可以对替换策略添加更多的控制，并且我们计划发出可以由外部客户端用于实现任意复杂替换和/或规模的事件下调政策。
复制控制器永远局限于这一狭窄的责任。它本身不会执行就绪和活性探测。不是执行自动缩放，而是要由外部自动缩放器（如＃492中所讨论的）来控制，这将更改其副本字段。我们不会向复制控制器添加调度策略（例如，扩展）。也不应该验证控制的pod与当前指定的模板匹配，因为这将阻碍自动调整大小和其他自动化过程。类似地，完成期限，顺序依赖性，配置扩展和其他特征属于其他地方。我们甚至计划散装大容量创建机制（＃170）。
复制控制器旨在是一个可组合的构建块原语。我们期望在其上构建更高级别的API和/或工具，以及为将来用户方便的其他补充原语。 kubectl目前支持的“宏”操作（运行，停止，缩放，滚动更新）是这方面的概念验证实例。例如，我们可以想象像Asgard管理复制控制器，自动标量，服务，调度策略，金丝雀(canary deployments:局部更新)等

#####API对象

复制控制器是kubernetes REST API中的顶级资源。有关API对象的更多详细信息，请参见：ReplicationController API对象。
#####Replication Controller的替代方法

#####Replica Sets

ReplicaSet是支持新的基于集合的标签选择器的下一代Replication Controller。它主要用于部署作为协调pod创建，删除和更新的机制。请注意，除非需要自定义更新编排或根本不需要更新，否则我们建议使用部署而不是直接使用副本集。

#####deployment（推荐）

部署是一个更高级别的API对象，以类似kubectl滚动更新的方式更新其底层副本集及其Pods。如果您想要此滚动更新功能，建议使用部署，因为与kubectl滚动更新不同，它们是声明式的，服务器端的，并且具有其他功能。
裸露的荚

与用户直接创建pod的情况不同，复制控制器替换因任何原因而被删除或终止的pod，例如在节点故障或破坏性节点维护（例如内核升级）的情况下。因此，我们建议您使用复制控制器，即使您的应用程序只需要一个单元。想象它类似于过程主管，只有它监督多个节点上的多个pod，而不是单个节点上的单个进程。复制控制器将本地容器重新启动委派给节点上的某个代理（例如Kubelet或Docker）。
工作

对预期自己终止（即批处理作业）的pod使用Job而不是复制控制器。
#####DaemonSet

对提供机器级功能（例如机器监视或机器日志记录）的pod使用DaemonSet而不是复制控制器。这些pod具有与机器生存期相关的生命期：pod需要在其他pod启动之前在机器上运行，并且当机器否则准备重新启动/关闭时可以安全地终止
