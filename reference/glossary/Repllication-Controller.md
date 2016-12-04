### Replication Controller

######什么是Replication Controller
Replication Controller简称副本控制器，它能保证在任何时候，都运行用户期望数量的pod。如果pod过多，则会杀死多余的pod，如果pod过少，则会自动增加pod，以达到用户期望的数量的pod。与手动创建的pod不同，如果复制控制器维护的pod失败，被删除或终止，它们将被自动替换。例如，在中断性维护（例如内核升级）之后，您的pod会在节点上重新创建。因此，我们建议您使用复制控制器，即使您的应用程序只需要一个单元。您可以将复制控制器看作类似于流程管理程序的东西，而不是单个节点上的单个进程，复制控制器监控多个节点上的多个pod。
复制控制器在讨论中通常缩写为“rc”或“rcs”，并且作为kubectl命令中的快捷方式。
一个简单的情况是创建1个Replication Controller对象，以便可靠地无限期地运行Pod的一个实例。更复杂的用例是运行复制服务的多个相同副本，例如Web服务器等

######运行一个rc的demo
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
