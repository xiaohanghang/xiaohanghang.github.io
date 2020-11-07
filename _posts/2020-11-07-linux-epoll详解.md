---
layout:     post
title:      epoll详解-linux
date:       2020-11-06 12:16:00
summary:    linux
categories: jekyll pixyll
---

## 进程的阻塞
> 正在执行的进程, 由于期待的某件事情未发生，如请求系统资源失败, 等待某个操作的完成，新数据还未到达, 则由系统自动执行阻塞, 使得自己由运行态变成阻塞状态.当进程进入阻塞状态时, 是不占用cpu资源的.

## 文件描述符
> 文件描述符在形式上是一个非整数，实际上, 它是一个索引值，指向内核为每个进程所维护的该进程打开文件的记录表.　


## IO模式
> 对于一次ＩＯ访问, 数据会被先拷贝到操作系统内核的缓冲区, 然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间, 所以当发生一个read操作时, 会经历两个阶段:
  - 等待数据准备
  - 将数据从内核拷贝到进程中

正是因为两个阶段, linux系统产生了下面五种网络模式的方案:
 - 阻塞io
 - 非阻塞io
 - io多路复用
 - 信号驱动io
 - 异步io

阻塞: 默认情况下所有socket都是blocking, 且执行过程中两个阶段都被阻塞.

非阻塞: 如果kernel中数据没有准备好，那么它不会block用户进程, 而是立刻返回一个error, 从用户进程角度来讲, 它发起一个read操作后, 并不需要等待, 而是立马得到一个结果.(需要主动询问kernel数据好了没有)

IO多路复用: select/epoll的好处在于单个process就可以同时处理多个网络连接的io.他的基本原理就是epoll这个function会不断论寻所有负责的socket, 当某个socket有数据到达时候, 就通知用户进程.
当用户进程调用select，　整个进程会被block，同时 kernel会监视所有的select负责的socket，当有socket数据准备好了, select就会返回, z这个时候用户进程再调用read操作, 将数据从kernel拷贝到用户进程.

select实际上使用了两个system call(select和recvfrom), 而blocking IO则只调用了一个system call(recv from), 但是用select的优势在于它可以同时处理多个连接, 但是如果连接数量不多, 可能延迟比多线程更慢, select/epoll的优势不在于对单个连接能处理的更快, 而是在于能处理更多的连接


