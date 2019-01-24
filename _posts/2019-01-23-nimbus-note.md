---
layout:     post
title:      numbus 笔记学习
date:       2019-01-23 20:51:00
summary:    numbus源码解析
categories: jekyll pixyll
---

> Nimbus 可以说是Storm最核心的部分，Nimbus本身是基于Thrift框架实现的，它使用了T的THsHaServer服务，THsHaServer即半同步半异步服务模式，它使用一个单独的线程来处理网络I/O,使用一个独立的工作者线程来处理消息，大大的提高消息的并发处理能力，主要的功能有两点：
- 对 Topology的任务进行分配调度
- 接收用户的命令并做出响应的处理，例如Topology的提交(submit)、杀死(kill)、激活和暂停以及重新调度。


### Nimbus相关的数据结构
Nimbus中用到的数据结构主要有两类：
- java定义的数据结构：任务分配
- Cloujure：Storm的元数据

### 下图为Nimbus中用到的几个Java数据结构间的类关系图
![](/images/djsjsjosw.jpg)

介绍图中各个对象的功能
- ExecutorDetails对象记录了每个Executor所对应的startTask和endTask，这样定义是为了保证Storm在计算Executor时，每个都Executor都是一个连续的任务集合。
- TopologyDetails对象记录每个Tolopogy的信息，包括topologyId,Topology使用过的配置项，以及每个Executor到所属组件的以及numWorkers等信息。
- SchedulerAssignmentImpl对象定义了当前Topology的任务分配情况，它包含topologyId以及为该Topology分配的<ExecutorDetails,WorkerSlot>映射关系。
- WorkSlot对象定义了一个可用资源，它包含nodeId和port两个成员变量，其中nodeId就是前面提到的supervisor-id，它实际上是指的某个Supervisor机器上的某个端口号。
- Cluster对象定义了集群当前的状态信息，包括所有的Supervisor信息以及当前所有的Topology的分配信息，它是任务调度器进行任务分配，调度的基础。

### Clojure数据结构
Nimbus使用Clojure语言定义了一些共享数据结构以及Storm元数据，其中最主要的数据结构是nimbus-data，Storm元数据包括StormBase，Assignment和SupervisorInfo。

1. nimbus-data介绍
nimbus-data数据结构定义了很多公用数据，其代码如下
![](/images/kddohdd.jpg)
