#### Impala 源码阅读 （一）
> 整理网路查询资料，总结Impala架构，以及各个组件的作用。

### Impala架构

> Impala使用去中心化的架构设计。


![架构](https://images2015.cnblogs.com/blog/183233/201603/183233-20160321223503417-887997697.png)

#### 主要组件
##### Impalad Daemon
`Impala`的核心组件是运行在每个节点上的`Impalad`这个守护进程`(Impalad Daemon)`，该程序负责读写数据，接受从`Impala-shell，Hue，JDBC`等客户端发送的查询请求。解析SQL生成执行计划，将执行计划分发的各个节点上并行计算，并负责将当前节点计算好的查询结果发送到协调器节点`(coordinator node)`

`Impala`使用`round-robin`算法实现负责均衡，将任务提交到不同的节点上。任务提交到哪个节点，该节点会被作为这次查询的协调器，其他节点会传输结果集到这个协调器节点。

`Impala Daemon`定期的跟`statestore`进行通信，确定哪些节点是健康的能够接收新的任务，并且获取最新的元数据信息，更新本地存放的元数据缓存信息。
> 当集群中的任意节点create、alter、drop任意对象、或者执行insert、load data的时候触发广播消息。

##### Impala Statestore
`Impala Statestore`检查集群中各个节点上`Impala Daemon`的健康状态，同时定期的向`Impalad`节点更新最新的节点状态数据，`Impalad`节点会缓存该数据。在整个集群中仅需要一个`statestore`节点。

`StateStore`进程是单点的，并且不会持久化任何数据到磁盘，如果服务挂掉，`Impalad`则依赖于上一次获得元数据状态进行任务分配。

如果某个`Impalad`节点由于硬件错误、软件错误或者其他原因导致离线，`statestore`就会通知其他的节点，避免其他节点再向这个离线的节点发送请求。

> 各个组件会在StateStored里订阅某个Topic，目前已知的Topic有：
> * impala-membership ：负责全局广播每个Impalad节点的进程健康状态，各Impalad都订阅了这个Topic，所以StateStored会定期发送这个Topic的心跳，广播所有节点的健康信息，也从心跳的Response得到所有节点的健康状态。
> * catalog-update：负责广播元数据的更新，Catalogd和各Impalad都订阅了这个Topic。所以StateStored会定期发送这个Topic的心跳，Catalogd收到这个心跳后会在Response里放入更新的表元数据，StateStored收到更新后会放入下一次广播的心跳里，Impalad收到心跳后会用更新的元数据更新本地的元数据信息。
> * impala-request-queue：负责广播每个Pool占用和Queue的情况，各Impalad都订阅了这个Topic。

##### Impala Catalog
`Impala Catalog`提供了元数据的服务。

它以单点的形式存在，它既可以从外部系统（如`Hive Metastore`）拉取元数据，也负责在`Impala`中执行的DDL语句提交到`Metatstore`，由于`Impala`没有`update/delete`操作，所以它不需要对`HDFS`做任何修改。

元数据是通过`StateStore`服务广播分发到每个`Impala`节点的，并且每个`Impala`节点在本地会缓存所有元数据。

整个集群中仅需要一个这样的进程。由于它的请求会跟`statestore daemon`交互，所以最好让`statestored`和`catalogd`这两个进程在同一节点上。
> catalog服务减少了refresh和invalidate metadata语句的使用。在之前的版本中，当在某个节点上执行了create database、drop database、create table、alter table、drop table语句之后，需要在其它的各个节点上执行命令invalidate metadata来确保元数据信息的更新。同样的，当你在某个节点上执行了insert语句，在其它节点上执行查询时就得先执行refresh table_name这个操作，这样才能识别到新增的数据文件。


### 源码结构
源码分成了两部分：
* 前端代码(FE)：前端代码由JAVA实现完成。
* 后端代码(BE)：后端代码由C++实现完成。

从架构图中，我们可以看出，`Impalad`组件是由`Planner, Coordinator, Exec Engine`三部分构成：
* Planner：负责解析查询请求，并生成执行计划树，属于前端代码。
* Coordinator：拆解请求（Fragment），负责定位数据位置，并发送请求到Exec Engine，汇聚请求结果上报，属于后端代码。
* Exec Engine：执行Fragment子查询，属于后端代码。

> 前后端代码关系，后端C++代码通过`JNI`调用前端JAVA代码。
> 前端解析SQL查询语句，生成查询计划树，再通过调度器把执行计划分发给具有相应数据的其它Impalad进行执行。
> Impalad节点读写数据，并行执行查询，并把结果通过网络流式的传送回给Coordinator，由Coordinator返回给客户端。
