---
layout:     post
title:      storm code
date:       2018-12-20 17:26:00
summary:    this is about storm code  concept
categories: jekyll pixyll
---

### 1.总体架构与代码结构


### 2.

这个类定义了一个Spout，它继承自BaseRichSpout，BaseRichSpout是一个实现了IRichBolt接口的虚类，它的nextTuple方法随机的从句子数组中选出一个句子发送出去，declareOutputFields方法申明了该Spout输出的消息模式，这里输出只有一列，字段名是word：



### 3.Fields 定义
Fields数据结构用于存储消息的字段名列表，其所需参数是字段名集合。对于同一条消息，在构建Fields对象时会为其所有的字段建立索引。它的定义如下：
<blockquote>
  <p>
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

  </p>
</blockquote>

<blockquote>
  <p>
  private void index(){
    for(int i = 0;i<_fields.size();i++){
        _index.put(_fields.get(i),i);
    }
  }
  </p>
</blockquote>


1. 第一行表明Fields类实现了Iterable<String>和Serializable。接口Iterable<String>定义了一个迭代器接口，用于遍历Fields中存储的字段名列表，接口Serializable则表明该类是可以被序列化的

2. 第2-3行分别定义了一个保存所有字段名的列表，以及保存了从字段名到它在字段名列表中位置的映射表。


### 4.Tuple 接口
Tuple是storm中的主要数据结构，在storm发送接收消息的过程中，每一条消息实际上都是一个Tuple对象，下面首先看一下Tuple接口的定义：

##   通信机制
### 分布式模式下实现

![](/images/5dfb0cbd211be2227a93ca1b31eb17b.png)
>  在分布式模式下，Strom采用ZMQ来进行通信,成员变量socket表示ZMQ的连接字符串。bb为预先分配的ByteBuffer空间，用来存储已被序列化的端口号。 ZMQ会分两次发送消息：第一次发送TaskId，第二次则为具体的消息内容，接收时同样先接收TaskId，然后接收具体内容。

![](/images/15470994851111.jpg)
- context为ZMQContext对象
- linger-ms 表示调用ZMQ Socket的关闭方法term后未发送消息的等待时间。若超出该事件，未发送的消息将被丢弃。该变量的默认值为5秒。
- hwm表示ZMQ发送队列的高水平线。若发送队列里面的消息个数超过hwm，新来的消息可能会被丢弃。
- local? 表示系统是否运行在单机环境下ZMQ的Socket参数。
- bind方法设置了ZMQ的Socket参数，注意这里的socket模式设置为pull类型，表示返回的Socket主要用于接收消息。
