---
layout:		post
author:		Essviv
title:		强引用，软引用，弱引用以及虚引用
subtitle:	含义与区别
date:		2016-04-16 17:55:00+0800
tag:		引用类型，GC
---

1. 强引用(Strong Reference)是指平时经常用到的引用类型，如果某个对象存在强引用，那么它将不会被GC回收

2. 软引用(Soft Reference)是指那些有用但不是必需的对象，它经常被用作缓存，当JVM内存充足时，它不会被GC回收，但如果内存不足时，它会被回收；它可以和引用队列(ReferenceQueue)进行关联，当软引用被回收时，它就进入关联的引用队列

3. 弱引用(Weak Reference)是指不是必需的引用，在GC开始的时候，不管内存是否充足，它都将被回收；它可以和引用队列（ReferenceQueue)相关联，当弱引用被回收时，它就被加入到相关联的引用队列中

4. 虚引用(Phantom Reference) 并不影响对象的生命周期，如果一个对象和虚引用关联，那么就跟没有和引用关联一样，它随时可能被GC回收，它必须和引用队列相关联，当GC某个对象时，如果发现它还有虚引用，则会把它加入到相应的引用队列中，可以通过判断虚引用是否出现在这个引用队列中，来确定该对象是否被回收

参考文献：[参考文献](https://community.oracle.com/blogs/enicholas/2006/05/04/understanding-weak-references)