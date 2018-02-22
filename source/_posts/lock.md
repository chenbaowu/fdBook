---
title: synchronized ， sleep ，wait ，notify，等的理解
date: 2017-05-13 21:21:21
categories: android
tags: 多线程
---

1.sleep是Thread类的静态方法，谁调用谁去睡觉。sleep是占用cpu去睡觉，而wait是放弃cpu去睡觉， sleep没有释放锁，而wait释放了锁，sleep不会让出cpu资源，wait是进入线程池等待，一般wait是不会使用时间参数，他必须等待别人notify他才会进入就绪队中。而sleep只要时间到了，就会自动进入就绪队列。如果等不及了，只能通过interrupt来强项打断。

2.wait，notify以及notifyall`只能在同步控制方法或者同步控制块中使用`，而sleep可是在任何地方使用。

3.sleep必须捕获异常，而其他3个则不需要。

<!-- more -->

4.在JAVA中的Object类型中，都是带有一个内存锁的，在有线程获取该内存锁后，其它线程无法访问该内存，从而实现JAVA中简单的同步、互斥操作。明白这个原理，就能理解为什么synchronized(this)与synchronized(static XXX)的区别

5.synchronized就是针对内存区块申请内存锁，this关键字代表类的一个对象，所以其内存锁是针对相同对象的互斥操作，而static成员属于类专有，其内存空间为该类所有成员共有，这就导致synchronized()对static成员加锁，相当于对类加锁，也就是在该类的所有成员间实现互斥，在同一时间只有一个线程可访问该类的实例。如果只是简单的想要实现在JAVA中的线程互斥，明白这些基本就已经够了。但如果需要在线程间相互唤醒的话就需要借助Object.wait(), Object.nofity()了

6.Obj.wait()，与Obj.notify()`必须要与synchronized(Obj)一起使用`，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制

7.wait()、notify()、notifyAll()是三个定义在Object类里的方法，可以用来控制线程的状态。
这三个方法最终调用的都是jvm级的native方法。随着jvm运行平台的不同可能有些许差异。
    如果对象调用了wait方法就会使持有该对象的线程把该对象的控制权交出去，然后处于等待状态。
    如果对象调用了notify方法就会通知某个正在等待这个对象的控制权的线程可以继续运行。
    如果对象调用了notifyAll方法就会通知所有等待这个对象控制权的线程继续运行。
其中wait方法有三个over load方法：

wait()

wait(long)

wait(long,int)

wait方法通过参数可以指定等待的时长。如果没有指定参数，默认一直等待直到被通知。