---
title: Java高并发编程-1
date: 2019-11-01 17:22:31
tags:
- Java
---

> **什么是线程？**
> 线程是程序执行的一个路径，每一个线程都有自己的局部变量表、程序计数器以及各自的生命周期。

> **线程的生命周期**
> 线程的生命周期包括以下5个阶段：
> **
> NEW
> RUNNABLE
> RUNNING
> BLOCKED
> TERMINATED
> **
> **NEW 状态：**
> 在没有执行start之前的状态，表示线程对象被创建，仅此而已。和创建一个其他对象没有区别。
> **RUNNABLE状态：**
> 调用start方法后，JVM进程创建一个线程，但是仅仅是创建一个线程，并不代表线程被执行，此时的线程在等待CPU的调度。
> **RUNNING状态：**
> 此时的线程开始真正的执行逻辑代码，有一个知识点是一个处于RUNNING状态的线程事实上也是一个RUNNABLE状态的线程，反过来不成立。(我的理解是，正在执行的线程可以是在单核情况下运行的，这就可以说是线程在RUNNABLE和RUNNING之间切换，所以可以说是处于RUNNABLE。但是一个处于RUNNABLE的线程本身就就没有被运行，就不可以说是处于RUNNING状态。)RUNNING状态下的线程可以说是一个想成的必经状态，除非线程意外终止。否则无论是进入BLOCKED状态还是进入TERMINATED状态，都需要经过RUNNING状态。
> **BLOCKED状态：**
> 从RUNNING状态进入BLOCKED状态的几种情况：
> 1、调用了sleep或者wait方法，进入了waitSet中；
> 2、进行了阻塞的IO操作；
> 3、获取某个锁资源，从而加入了该锁的阻塞队列；
> **TERMINATED状态：**
> TERMINATED状态是线程的最终状态，该状态下不会切换到其他任何状态，因为此章台代表线程的生命周期已经结束。

> **线程的start方法——模板模式在Thread中的应用**
> start方法的源码：

```
publi c  synchronized void start() {
     if (threadStatus != 0) #检查线程状态
     throw new IllegalThreadStateException(); 
     group.add(this); 
     boolean started = false; 
     try { 
         startO();
          started = true; 
          } finally { 
              try {
                   if (!started) {
                        group.threadStartFailed(this); 
                        } 
                        catch (Throwable ignore) {
                        }
              }
          }
```

> 核心函数start0是一个本地方法：

```
private native void start0();
```

> 从上面的代码可以看出，一个线程是不可以start两次的，不然会报非法线程异常。同时一个生命周期结束的线程在调用start方法是也会报出相同的异常。同时还可以知道线程的真正逻辑在run方法中，这里的run函数就相当于模板模式中需要重写的函数，而start函数就是模板模式中的模板函数。这样做的好处是将线程的控制和业务的执行分离开来。

> **Runnable和Thread的关系及使用**
> 网上看了好多资料，最后还是没有找到一个比较有说服力的答案，最后没有办法。我打开了Thread的源码一探究竟。

```
//开头是一些版权信息，接下来是一大段介绍摘抄几句比较重要的：

 Each thread
 * may or may not also be marked as a daemon. When code running in
 * some thread creates a new <code>Thread</code> object, the new
 * thread has its priority initially set equal to the priority of the
 * creating thread, and is a daemon thread if and only if the
 * creating thread is a daemon.
 （貌似说的是线程被创建的时候会被设置默认优先级，默认的优先级是和创建线程的优先级是一样的）

 * When a Java Virtual Machine starts up, there is usually a single
 * non-daemon thread (which typically calls the method named
 * <code>main</code> of some designated class). The Java Virtual
 * Machine continues to execute threads until either of the following
 * occurs:
 * <ul>
 * <li>The <code>exit</code> method of class <code>Runtime</code> has been
 *     called and the security manager has permitted the exit operation
 *     to take place.
 * <li>All threads that are not daemon threads have died, either by
 *     returning from the call to the <code>run</code> method or by
 *     throwing an exception that propagates beyond the <code>run</code>
 *     method.
 * </ul>
 （JVM启动时只有一个非守护线程，这些线程一直被JVM执行直到一下几种情况：
    *安全退出
    *非守护线程全部死亡
    *引起异常）

* There are two ways to create a new thread of execution. One is to
 * declare a class to be a subclass of <code>Thread</code>. This
 * subclass should override the <code>run</code> method of class
 * <code>Thread</code>. An instance of the subclass can then be
 * allocated and started. For example, a thread that computes primes
 * larger than a stated value could be written as follows:
 * <hr><blockquote><pre>
 *     class PrimeThread extends Thread {
 *         long minPrime;
 *         PrimeThread(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeThread p = new PrimeThread(143);
 *     p.start();
 * </pre></blockquote>
 * <p>
 （有两种方法可以创建一个线程，一是继承Thread类，重写run方法并给出了实例函数）

 * The other way to create a thread is to declare a class that
 * implements the <code>Runnable</code> interface. That class then
 * implements the <code>run</code> method. An instance of the class can
 * then be allocated, passed as an argument when creating
 * <code>Thread</code>, and started. The same example in this other
 * style looks like the following:
 * <hr><blockquote><pre>
 *     class PrimeRun implements Runnable {
 *         long minPrime;
 *         PrimeRun(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeRun p = new PrimeRun(143);
 *     new Thread(p).start();
 * </pre></blockquote>
 * <p>
 * Every thread has a name for identification purposes. More than
 * one thread may have the same name. If a name is not specified when
 * a thread is created, a new name is generated for it.
 * <p>
 * Unless otherwise noted, passing a {@code null} argument to a constructor
 * or method in this class will cause a {@link NullPointerException} to be
 * thrown.
 （另外一方法是实现Runnable接口，并给出了实现方法，需要注意的是两种方法的启动线程的方式是不一样的。）
```

其实从上面的介绍就可以看出来，Thread和Runnable是没有本质区别的，只不过按照OOP来说一个是具体的实现，一个是接口的规范。只是两种实现线程的不同操作。
接下来是几个比较重要的函数：

```
//用来生成线程ID的整型变量，为了保证唯一性添加synchronized同步锁。
 /* For autonumbering anonymous threads. */
private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }

//可以看到前面说到的start两次线程失败是因为有这个参数，同时volatile参数是来保持其可见性，当其被改变时调用此线程的对象可以立即感知到。
private volatile int threadStatus = 0;

//属性中有target，解释为将要被run的对象。
/* What will be run. */
    private Runnable target;

//Thread的构造函数太多了，不一一罗列。其中有Runnable target的上面的target属性就会被赋值，结合下面的run方法就可以知道Thread执行的也是Runnable的方法。

@Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

一圈看下来，发现Thread是一个完整的类，提供了丰富的接口函数实现，通过继承Thread实现的线程对象可以有丰富的功能。利用Runnable实现的线程最终还是要放进Thread中去启动，只不过可以设计自己的业务逻辑而不必继承Thread的方法。