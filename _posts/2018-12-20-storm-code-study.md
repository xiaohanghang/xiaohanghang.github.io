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
