---
layout:     post
title:      "Java 并发 篇一"
subtitle:   "Java Cocurrency Part 1"
date:       2019-06-01 22:40:00
author:     "jk"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - Java
    - Cocurrency
---
## ReadWriteLock

读锁为共享锁，但排斥写锁；写锁为排他锁

### 思考：缓存系统

用户和数据库中间的一个环节，用户直接访问数据库的时间是远大于直接访问内存，有了缓存区后用户访问数据时 ，先访问缓存区当缓存区有用户需要的数据时直接拿走，当缓存区没有这样的数据，访问数据库并把访问所得的数据放在缓存区，这样当下一个需要这个数据的用户就直接访问内存即可得到。

实现：使用读写锁实现，参考API文档中的例子修改

```java
class CachedData {
   Object data;
   volatile boolean cacheValid;
   final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
   void processCachedData() {
     rwl.readLock().lock();  //1. 上读锁
     if (!cacheValid) {      //2. 验证cacheValid
        // Must release read lock before acquiring write lock
        rwl.readLock().unlock(); //3. 解除读锁
        rwl.writeLock().lock(); //4. 上写锁
        try {
          // *** Recheck state because another thread might have
          // acquired write lock and changed state before we did.
          if (!cacheValid) {    //5. 验证cacheValid
            data = ...
            cacheValid = true;
          }
          // Downgrade by acquiring read lock before releasing write lock
          rwl.readLock().lock(); //*** 6. 上读锁 锁降级
        } finally {
          rwl.writeLock().unlock(); // Unlock write, still hold read //7. 解除写锁
        }
     }
     try {
       use(data);
     } finally {
       rwl.readLock().unlock();//8. 解除读锁
     }
   }
 }
```

### 源码剖析

[【死磕 Java 并发】—– J.U.C 之读写锁：ReentrantReadWriteLock](http://www.iocoder.cn/JUC/sike/ReentrantReadWriteLock/)

#### 同步状态

在 ReentrantLock 中，使用 Sync ( 实际是 AQS )的 int 类型的 state 来表示同步状态，表示锁被一个线程重复获取的次数。但是，读写锁 ReentrantReadWriteLock 内部维护着一对读写锁，如果要用一个变量维护多种状态，需要采用“按位切割使用”的方式来维护这个变量，将其切分为两部分：高16为表示读，低16为表示写。

分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过位运算。假如当前同步状态为S，那么：

写状态，等于 S & 0x0000FFFF（将高 16 位全部抹去）
读状态，等于 S >>> 16 (无符号补 0 右移 16 位)。

```java
// Sync.java

static final int SHARED_SHIFT   = 16; // 位数
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; // 每个锁的最大重入次数，65535
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

同步状态的低16位用来表示写锁的获取次数，同步状态的高16位用来表示读锁被获取的次数。

![ReadWriteLock](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/ReadWriteLock.png)

### 主要特性

- 支持公平性
- 支持重入性，读写分别支持65535个递归重入锁
- 支持锁降级——遵循获取写锁，再获取读锁，最后释放写锁的次序，如此写锁能够降级称为读锁

## Condition

Condition是显示锁提供的一个同步工具，它的使用也依赖显示锁。通过`lock.newCondition()`创建一个Condition对象，该方法实际上是new出一个ConditionObject对象。Condition是依赖显示锁使用的，而显示锁又是通过AQS实现的，所以其实上述的ConditonObject是AQS的一个内部类，该类实现了Conditon接口。

### 思考：环形阻塞缓冲区(Buffer)

```java
public class ConditionBuffer<T> {
    private Lock lock = new ReentrantLock();
    private Condition empty = lock.newCondition();
    private Condition full = lock.newCondition();
    private Object[] items;
    private int count = 0;
    private int head = 0;
    private int tail = 0;

    public ConditionBuffer (int cap) {
        items = new Object[cap];
    }

    public void put (T item) throws InterruptedException {
        lock.lock();
        try{
            while(count == items.length) {
                System.out.println("This Buffer is full");
                full.await();
            }
            items[tail] = item;
            if(++tail == items.length) {
                tail = 0;
            }
            ++count;
            System.out.println(count);
            empty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T take () throws  InterruptedException {
        lock.lock();
        try {
            while(count == 0) {
                System.out.println("This Buffer is empty");
                empty.await();
            }
            T item = (T)items[head];
            items[head] = null;
            if(++head == items.length) {
                head = 0;
            }
            count--;
            System.out.println(count);
            full.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### 源码剖析

[**详解Condition的await和signal等待通知机制**](https://github.com/CL0610/Java-concurrency/blob/master/12.%E8%AF%A6%E8%A7%A3Condition%E7%9A%84await%E5%92%8Csignal%E7%AD%89%E5%BE%85%E9%80%9A%E7%9F%A5%E6%9C%BA%E5%88%B6/%E8%AF%A6%E8%A7%A3Condition%E7%9A%84await%E5%92%8Csignal%E7%AD%89%E5%BE%85%E9%80%9A%E7%9F%A5%E6%9C%BA%E5%88%B6.md)

[AQS源码分析之ConditionObject](https://blog.csdn.net/luofenghan/article/details/75069065)

ConditonObject维护了两个队列

- Lock.Sync中的同步队列，队列中是等待锁的线程
- condition中的等待队列，队列中是当前Condition中等待唤醒的队列。

await方法中，当前线程是持有锁的，所以需要先释放锁，然后通过一个循环判断当前线程有没有存在同步队列中作为是否唤醒的条件，因为signal方法会将等待队列中的相应线程移动到同步队列中，线程唤醒后在同步队列中继续获取锁。

### 主要特性

- Condition依赖于显示锁，wait/notify/notifyall依赖传统的对象内置锁
- 一个显示锁可以生成多个条件队列，每个条件队列可以对应一个条件谓词，而内置对象锁只能有一个条件队列，一个条件队列对应多个条件谓词，导致notifyall可能会唤醒不相干的条件谓词
- Condition支持超时设置

## ConcurrentHashMap

### HashMap并发 cpu 100%

HashMap 多线程并发导致cup100%的原因，参考 [老生常谈，HashMap的死循环](https://juejin.im/post/5a66a08d5188253dc3321da0)，简单的说就是在并发扩容时，多线程操作容易在一个桶上形成环，当扩容完成，执行get操作时出现死循环。

### Diff 1.7 & 1.8

HashMap和ConcurrentHashMap区别参考 [HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！](https://juejin.im/post/5b551e8df265da0f84562403#heading-13)

![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/hashmap.png)
![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/hashmap18.png)
![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/cocurrenthashmap.png)
![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/concurrenthahmap1.png)

### Q ?

- CAS 和 Synchronized的具体用处在哪里？参考上述文章

- 为什么不都用CAS？ - CAS只能确保变量但是不能做块的复合操作

- Synchronized && Lock 

  [ConcurrentHashMap 1.8为什么要使用CAS+Synchronized取代Segment+ReentrantLock](https://www.cnblogs.com/yangfeiORfeiyang/p/9694383.html)

  Synchronized相比较lock少一些上下文切换的开销

### size

参考 [并发编程 —— ConcurrentHashMap size 方法原理分析](https://juejin.im/post/5ae75584f265da0b873a4810)

size方法仍然不准，因为执行size的时候，可能其他线程在并发做CAS操作更新baseCount或者CounterCell 

## ConcurrentLinkedQueue

ConcurrentLinkedQueue的offer和poll方法的head和tail是延迟更新的。，在判断队列是否为空队列的时候是不能通过线程在poll的时候返回为null进行判断的，可以通过isEmpty方法进行判断。

思考ConcurrentLinkedQueue和BlockingQueue的使用场景区别。

## CopyOnWriteArrayList

CopyOnWriteArrayList 主要的思想是写时复制，即COW思想。COW思想牺牲了数据实时性，只保证数据的最终一致性。

### 源码剖析

- get方法从代码层面看，就是“单线程代码”， 没有线程安全，没有加锁，没有CAS，所有读操作不会修改容器中的内容。
- add方法
  - 使用Lock，保证同一时刻只有一个写线程，避免出现多个写副本。
  - 数组引用是volatile，保证写线程最终对引用的修改对读线程可见。
  - 读写线程分别操作不同的数组副本，不会发生冲突。

### COW vs RW LOCK

读写锁，写锁是排他锁，写锁会阻塞读锁；COW的写方法与读方法并行，不会发生阻塞

### CopyOnWriteArrayList 缺点

- 内存占用
- 牺牲数据一致性

## BlockingQueue

![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/blockingqueue.png)

### ArrayBlockingQueue

ArrayBlockingQueue 内部采用`数组`存储数据，利用`Condition机制`实现可阻塞式通信

### LinkedBlockingQueue

LinkedBlockingQueue是内用采用`链表`实现的`有界阻塞`队列，LinkedBlockingQueue与ArrayBlockingQueue内部同样采用Condition机制实现通信，但是LinkedBlocking内部采用了锁分离思想，使用了两把锁分别生成两个Condition来进行通信。

#### LinkedBlockingQueue源码剖析

- LinkedBlockingQueue中放入数据阻塞的时候，因为它内部有2个锁，可以并行执行放入数据和消费数据，不仅在消费数据的时候进行唤醒插入阻塞的线程，同时在插入的时候如果容量还没满，也会唤醒插入阻塞的线程
- 链表队列的添加和头部的删除，都是只和一个节点相关，添加只往后加就可以，删除只从头部去掉就好。为了防止head和tail相互影响出现问题，这里就需要原子性的计数器，头部要移除，首先得看计数器是否大于0，每个添加操作，都是先加入队列，然后计数器加1，这样保证了，队列在移除的时候，长度是大于等于计数器的，通过原子性的计数器，双锁才能互不干扰
- 需要注意的是remove方法由于要删除的数据的位置不确定，需要2个锁同时加锁

> 数组的一个问题就是位置的选择没有办法原子化，因为位置会循环，走到最后一个位置后就返回到第一个位置，这样的操作无法原子化，所以只能是加锁来解决。

## ThreadLocal

![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/threadlocal.png)

每个Thread有一个ThreadLocalMap变量，ThreadLocalMap是ThreadLocal的一个静态内部类，每个ThreadLocal通过当前Thread对象可以获取Thread对应的ThreadLocalMap，ThreadLoaclMap内的Entry以当前ThreadLocal实例为key进行存储。Entry继承了弱引用类，所以Entry中的key，即对应的ThreadLocal为弱引用，使用完当前ThreadLocal后调用remove方法进行清理。

## 线程池

![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/threadpool.png)
![](https://img.charflow.com/2019-06-01-Java_Cocurrency_One/executor.png)
