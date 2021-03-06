---
layout:		post
title:		java集合学习
subtitle:	并发实现
date:		2016-04-18 21:55:00+0800
tags:		java collection implement concurrent
---

JAVA的集合框架中提供了很多通用的实现，但这些实现基本上都不是线程安全的，因此，JAVA框架还提供了这些集合类的并发实现，这篇文章专门来讨论这些并发的实现

# BlockingQueue接口

这个接口定义了线程安全的QUEUE接口，在从队列里读取或者插入操作时，如果队列为空，或者已经到达队列上限，那么操作会被阻塞. 按操作无法立即执行时方法执行的操作可以将BlockingQueue的方法分为四类：

* 抛出异常: add, remove, element

* 返回特殊值: offer, poll, peek

* 阻塞直到可执行: put, take

* 阻塞一定时间: offer, poll

BlockingQueue的实现有五种： 

* ArrayBlockingQueue: BlockingQueue的数组实现，这种实现的元素在创建时就定好了，如果达到数组的上限，那么offer和put操作将不能得到立即实现;

* LinkedBlockingQueue: 阻塞队列的链表实现. 这种实现默认是是没有元素个数限制的, 当然如果希望有上限的话，也可以设置

* DelayQueue: 延迟队列，它的元素必须实现了Delay接口，返回延迟的时间，在这个时间前，元素不能被消费,其底层是通过PriorityQueue来实现的.

* PriorityBlockingQueue: 这种实现可以是PriorityQueue的并发实现, 它的所有元素必须实现Comparable，或者在构造对象时提供相应的comparator接口的实现，这点和PriorityQueue是一致的. 

* SynchronizedQueue: 这是一种特殊的阻塞队列的实现，它只有一个元素，并且插入的线程一直阻塞到这个元素被消费了为止；同样地，如果一个线程试图从空队列中取元素，那么它也会一直阻塞到有其它线程往这个队列里插入新的元素为止. 

# BlockingDeque接口

这个接口是双向队列的阻塞接口，它和BlockingQueue的关系就像Queue和Deque的关系是一样的，所有的BlockingQueue的操作在双向扩展后即是BlockingDeque的方法，这里就不做具体的叙述，实现这个接口的类只有一个**LinkedBlockingDeque**,从名字可以看出，这是一种链表实现.

# ConcurrentMap接口

这是Map的并发实现接口, 它提供了线程安全的访问Map的方法，它的实现有**ConcurrentHashMap**, 这个实现和HashTable十分相似，但有几点不同：

* 在进行读操作时，ConcurrentHashMap不会对map加锁，而hashTable会对所有的操作进行同步

* 在进行写操作时，concurrentHashMap不会对整个map加锁，它将整个map分为几个部分（默认为16）,每次写操作时只会对一个部分进行加锁，而hashTable会对整个map进行同步

* 在迭代过程, 如果map的内容发生了变化，concurrentHashMap不会抛出异常，虽然它的迭代器并不是为并发设计的

# 参考文献

Sample代码： [这里](https://github.com/Essviv/collections)

参考文献: [参考文献](http://tutorials.jenkov.com/java-util-concurrent/index.html)