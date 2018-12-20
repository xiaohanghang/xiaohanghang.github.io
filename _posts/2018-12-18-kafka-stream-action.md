---
layout:     post
title:      Kafka streams
date:       2018-12-18 11:55:00
summary:    this is about kafka streams basic concept
categories: jekyll pixyll
---

## kafka streams

####  什么是流处理
- Gathering information per sale for a given customer implies that all transactions for customer are on the same partition. But the transactions come into the application without a key,the producer assigns the transactions to partitions in a round-robin fashion.Because the key is not populated,round-robin assignment means the transactions for a given customer won't land on the same partition.


- Placing customer transactions with the same ID on the same partition is important,bacause you need to look up records by ID in the state store.Otherwise, you'll have customers with the same ID spread across different partitions,requiring you to look up the same customer in multiple state stores.(This statement could be interpreted to mean that each partition has its own state store,but that's no the case. partitions are assigned to a streamsTask,and each streamsTask has its own state store)


#### Repartitioning the data
let's have a general disscussion on how Repartitioning works,To Repartitioning records.
    1. first you may modify or change the key on the original record,and then you write out records to a new topic.
    2. you consume those records again;but as a result of Repartitioning,those records may come from different partitions than they were in originally.
##### Repartitioning in kafka streams
Repartitioning in kafka streams is easily accomplished by using the KStream.through(),it creates an intermediate topic,and the current KStream instance will start writing records to that topic.A new KStream instance is returned from the through(),using the same intermediate topic for its source.This way,the data is **seamlessly Repartitioning**

KStream.through() takes two parameters: the topic name and a Produced instance that provides the key Serde, and a streams-partitioner.

<blockquote>
  <p>

    RewardStreamPartitioner streamPartitioner = new RewardStreamPartitioner();
    KStream<String, Purchase> tranByCustomerStream = PurchaseKStream.through("customer_transactions",Produced.with(stringSerde,PurchaseSerde,streamPartitioner));


    public class RewardStreamPartitioner implements StreamsPartitioner<String,Purchase>{
        public Integer partition(String key,Purchase value,int numPartitions){
          return value.getCustomerId().hashCode() % numPartitions;      
        }
    }
  </p>
</blockquote>
Notice that you haven't generated a new key,You're using property of the value to determine the correct partition.The key point to take away from this quick detour is that when you're using state to update and modify records,it's necessary for those records to be on the same partition.
