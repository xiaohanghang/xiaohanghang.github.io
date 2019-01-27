---
layout:     post
title:      nimbus 笔记学习
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

2. Nimbus中的线程介绍
除了主服务器之外，Nimbus还有一个计数器线程，它的作用有3：
  - 调用 mk-assigment方法启动新一轮的任务分配，调用do-cleanup方法清理Storm元数据。
  - 调用 clean-inbox 方法清理Nimbus本地目录中Topology的jar包，该操作则每隔Nimbus-inbox-freq-secs(默认是600秒)执行一次。
  - 执行Topology的状态转移事件。只有当Nimbus接收到对应的服务请求(kill、reblance、activate和deactivate)时才会被触发。

#### mk-assigment
> mk-assigment方法主要负责对当前集群中所有Topology进行新一轮的任务调度。一方面检查已运行的Topology所占用的资源，判断是否存在问题，是否需要重新分配；另一方面根据系统当前的可用资源，为新提交的Topology分配任务。然后，mk-assignments方法会将所有的分配信息保存或者更新到ZooKeeper中，Supervisor会周期检测这些分配信息，并根据这些分配信息做出相关的调度处理。

#### Topology 状态转移
> Nimbus需要监视当前所有Topology的状态，并根据收到的Topology状态转移请求(kill/reblance/activate/e和deactivate)完成响应的状态转移，在Nimbus中，我们定义了通用的状态转移方法，他们会根据出传入的转移事件做相应的处理，这些方法包括transition-name！、transition！和state-transitions。

#### transition-name！
该方法会根据topology-name以及对应的转移事件完成Topology的状态转移，相关代码如下：
![](/images/sos0sy0s.jpg)

首先，它通过调用get-storm-id方法，为传入的storm-name查找对应的storm-id，如果找到了，就调用transition!方法。。所以它的作用实际上就是将基于storm-name的状态转换为基于storm-id的状态。

#### transition！
> transition!方法会根据传入的转移事件，获取与当前Topology状态对应的状态转移函数，并执行该函数取得转换后的新状态。如果新状态不为空，则将其更新到Zookeeper中。

![](/images/sljssjs0sa2.png)
 - 2-3行是一个重载的方法， 它会直接调用第4行的方法。默认的参数false表示在找不到对应的转换方式时将不抛出异常。
 - 第5行尝试获取nimbus-data的submit-lock。为了提高Nimbus的处理能力，在storm所采取的处理模式中(后面介绍的 state-transition)，无论是Nimbus的主服务线程还是计时器线程线程都会调用transition方法。因此为了确保逻辑的正确性，必须在这里做同步。
 - 12-23行定义了get-event方法，该方法会根据键从一个哈希表中查找对应的状态转移函数。如果没能找到，则将这一情况记录下来并根据error-on-no-transition的设置做相应的处理。
 - 24-26行首先调用state-transitions方法获取所有的状态跟状态间的转移事件(哈希表，键是原状态，值是一个从转移事件到状态转移函数的哈希表)，然后根据Topology的状态找到对应的从转移事件到状态转移函数的哈希表，最后调用get-event方法根据传入的转移事件获取状态转移函数。
