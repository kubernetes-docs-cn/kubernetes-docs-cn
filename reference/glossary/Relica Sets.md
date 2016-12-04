### Replica Sets

######什么是Replica Sets
Replica Sets是kubernetes中的下一代repllication-controller（副本控制器），rc所有的功能它都支持。它与rc的区别在于对标签选择器的支持不同，rc仅支持等级的标签选择器，例如：=  == !=。而Replica Sets 支持更加丰富的标签选择器操作，例如：in  not in等。

######什么时候用Replica Sets
replica sets确保在任何给定时间运行指定数量的pod。 但是，你可以用deployment管理replica set，并为pod提供声明性更新以及许多其他有用的功能。 因此，我们建议使用deployment，而不是直接使用replica set，除非您需要自定义更新编排或根本不需要更新。
这实际上意味着您可能永远不需要操作replica set：直接使用部署并在规范部分中定义您的应用程序
