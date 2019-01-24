---
layout:     post
title:      storm code
date:       2018-12-20 17:26:00
summary:    this is about storm code  concept
categories: jekyll pixyll
---

### 1.总体架构与代码结构
![](/images/sknssns.jpg)

Storm元数据：
![](/images/sjspsjs.jpg)

storm如何使用这些元数据：例如数据何时被写入，更新或者删除。这些数据都是哪种类型的节点来维护，关系网络的总体交互如下：
![](/images/lhsshsps.jpg)


![](/images/sohspshsp.jpg)
![](/images/ssoshsss.jpg.jpg)


箭头1由Nimbus创建的路径：
对于路径a，Nimbus只会创建路径，不会设置数据(数据由Worker设置)；对于b和c，Nimbus在创建它们的时候就会设置数据。a和b只有在提交新的Topology的时候才会创建，且b中的数据设置好后就不会再改变，c则在第一次为该Topology进行任务分配的时候创建，若任务分配有变，Nimbus就会更新它的内容。




### 2.

这个类定义了一个Spout，它继承自BaseRichSpout，BaseRichSpout是一个实现了IRichBolt接口的虚类，它的nextTuple方法随机的从句子数组中选出一个句子发送出去，declareOutputFields方法申明了该Spout输出的消息模式，这里输出只有一列，字段名是word：

### 3.Fields 定义
Fields数据结构用于存储消息的字段名列表，其所需参数是字段名集合。对于同一条消息，在构建Fields对象时会为其所有的字段建立索引。它的定义如下：
{% highlight java %}
public class Fields implements Iterable<String>,Serializable{
  private List<String> _fields;
  private Map<String,Integer> _index = new HashMap<String,Integer>();
  public Fields(String ... fields){
     this(Arrays.asList(fields));
  }
  public Fields(List<String> fields){
    _fields = new ArrayList<String>(fields.size());
    for(String field: fields){
      if(_fields.contains(field))
      throw new IllegalArgumentException();
    }
      _fields.add(field);
  }
  index();
  }
  }
  private void index(){
    for(int i = 0;i<_fields.size();i++){
        _index.put(_fields.get(i),i);
    }
  }
{% endhighlight %}

1. 第一行表明Fields类实现了Iterable<String>和Serializable。接口Iterable<String>定义了一个迭代器接口，用于遍历Fields中存储的字段名列表，接口Serializable则表明该类是可以被序列化的

2. 第2-3行分别定义了一个保存所有字段名的列表，以及保存了从字段名到它在字段名列表中位置的映射表。


### 4.Tuple 接口
Tuple是storm中的主要数据结构，在storm发送接收消息的过程中，每一条消息实际上都是一个Tuple对象，下面首先看一下Tuple接口的定义：

### 通信机制  分布式模式下实现

![](/images/5dfb0cbd211be2227a93ca1b31eb17b.png)
>  在分布式模式下，Strom采用ZMQ来进行通信,成员变量socket表示ZMQ的连接字符串。bb为预先分配的ByteBuffer空间，用来存储已被序列化的端口号。 ZMQ会分两次发送消息：第一次发送TaskId，第二次则为具体的消息内容，接收时同样先接收TaskId，然后接收具体内容。



![](/images/15470994851111.jpg)

- context为ZMQContext对象
- linger-ms 表示调用ZMQ Socket的关闭方法term后未发送消息的等待时间。若超出该事件，未发送的消息将被丢弃。该变量的默认值为5秒。
- hwm表示ZMQ发送队列的高水平线。若发送队列里面的消息个数超过hwm，新来的消息可能会被丢弃。
- local? 表示系统是否运行在单机环境下ZMQ的Socket参数。
- bind方法设置了ZMQ的Socket参数，注意这里的socket模式设置为pull类型，表示返回的Socket主要用于接收消息。


### supervisor守护进程的工作方式
> supervisor守护进程等待nimbus分配任务后生成并监控worker(JVM进程)执行任务，supervisor和worker都是运行在不同的jvm进程上，如果由supervisor拉起的一个worker进程因为错误(kill -9)异常退出，supervisor守护进程会尝试重新生成新的worker进程。

#### storm保障传输机制
> Strom的tuple锚定h和应答确认机制，当打开了可靠传输的选项，传输到故障节点上的tuple将不会收到确认应答，spout会因为超时而重新发送原始的tuple。这样的过程会一直重复直到topology从故障中恢复开始正常处理数据。


#### storm的可靠性
>  在storm中，可靠的消息处理机制是从spout开始的，一个提供了可靠的处理机制的spout需要记录它发射出去的tuple，当下游bolt处理tuple或者子tuple失败时，spout能够重新发射，子tuple可以理解为bolt处理spout发射的原始tuple，作为结果发射出去的tuple。另一个视角来看，可以将spout发射的数据流看成一个tuple树的主干。

![](/images/11111111.jpg)

在图中，实线部分为spout发射的原始主干tuple，虚线部分表示的子tuple都是源自于原始的tuple。这样产生的图形叫做tuple树，在有保障数据的处理过程中，bolt每收到一个tuple，都需要向上确认应答(ack)者报错。

bolt的可靠性：
1.当发射衍生的tuple时，需要锚定读入的tuple
2.当处理消息成功或者失败时分别确认应答或者报错。
锚定一个tuple的意思是，建立读入tuple和衍生出的tuple之间的对应关系，这样下游的bolt就可以通过应答确认、报错或者超时来加入到tuple树结构中。可以通过调用OutputCollector中emit()的一个重载函数锚定一个或者一组tuple：
collector.emit(tuple,new Values(word));
这里我们将读入的tuple和发射的新tuple锚定起来，下游的bolt就需要对输出的tuple进行确认应答或者报错，另外一个emit()方法发射非锚定的tuple：
collector.emit(new Values(word));
当处理完成或者发送了新的tuple之后，可靠的数据流中的bolt需要应答读入的tuple：
this.collector.ack(tuple);
