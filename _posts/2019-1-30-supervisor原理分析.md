---
layout:     post
title:      storm中的supervisor
date:       2019-01-30 15:30:00
summary:    supervisor中代码分析
categories: jekyll pixyll
---

> Supervisor可以理解为单机任务调度器，它负责监听Nimbus的任务调度，启动相应的Worker对Nimbus分配的任务进行处理。同时，它也会监听由它启动的worker的运行状态，一旦发现有worker处于非正常状态，就会杀掉该Worker，并将原来分配给该Worker的任务交还Nimbus进行重新分配。

### 与supervisor相关的数据结构
- standalone-supervisor方法：创建y一个实现了ISupervisor接口的对象，其中定义了一些常见的方法。
- supervisor-data：创建一个映射集合，包含很多会被其他方法共享的成员。
- Supervisor使用LocalState在本地保存了自己的ID信息，LocalAssigment信息以及有效的从Worker到端口号的映射。


### Supervisor中的线程
> Supervisor中有三个线程，一个计时器线程以及两个事件线程。计时器线程主要负责维持心跳，也即更新Zookeeper中保存的当前Supervisor的最新状态，
