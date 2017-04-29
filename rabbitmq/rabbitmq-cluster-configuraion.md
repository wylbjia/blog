# RabbitMQ 集群环境配置

**准备 Erlang Cookie**

1. Cookie 文件位置 /var/lib/rabbitmq/.erlang.cookie 或者当前用户 HOME 目录下 .erlang.cookie
2. 文件 .erlang.cookie 权限必须是 400
3. 将 .erlang.cookie 文件复制到集群所有节点上的相同位置
   
集群环境信息

节点名称 |    IP 地址    | 节点类型
---------|---------------|----------
node1    | 192.168.1.101 | 磁盘节点
node2    | 192.168.1.102 | 内存节点
node3    | 192.168.1.103 | 内存节点

**创建 RabbitMQ 集群**

停止节点上的应用   

```
$ node2> rabbitmqctl stop_app
```

重置节点元数据

```
$node2> rabbitmqctl reset
```

将节点加入到 node1 集群中，--ram 表示内存节点

```
$node2> rabbitmqctl join_cluster --ram rabbit@node1
```

启动节点上的应用

```
$node2> rabbitmqctl start_app
$node3> rabbitmqctl stop_app
$node3> rabbitmqctl reset
$node3> rabbitmqctl join_cluster --ram rabbit@node1
$node3> rabbitmqctl start_app
```

**将节点从集群中删除**

将 node3 节点从集群中删除，首先停止节点上的应用

```
$node3> rabbitmqctl stop_app
```

重置节点元数据

```
$node3> rabbitmqctl reset
```

启动节点上的应用

```
$node3> rabbitmqctl start_app
```

> 注意：加入到集群中的节点可在任何时候停止， 对于崩溃来说也没有问题. 在这两种情况下，集群剩余的节点将不受影响地继续操作，当它们重启的时候，这些崩溃的节点会再次自动追赶上其它的集群节点。
  
**删除一个无响应节点**

当 node3 节点无响应，从 node1 集群中强制删除 node3 节点

```
$node1> rabbitmqctl forget_cluster_node node3
```

> 注意：当 node3 节点它自己重新恢复后会认为它还在集群中，所以需要重置自己的状态，然后再加入到集群中.
  
**修改一个节点类型**

将 node2 内存节点变为磁盘节点，停止 node2 节点上的应用

```  
$node2> rabbitmqctl stop_app
```

将 node2 节点修改为磁盘节点

```
$node2> rabbitmqctl change_cluster_node_type disc
```

重启 node2 节点上的应用

```
$node2> rabbitmqctl start_app
```  
  
------------------------------------------------------------------------------------------

By typefo <typefo@qq.com> Update: 2017-04-29 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)
