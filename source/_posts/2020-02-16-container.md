---
title: 高并发与多线程 - Java容器
date: 2020-02-16 22:05:29
toc: true
categories:
- 技术笔记
tags: 
- Thread
- 容器
---
### Java 容器
容器 分两大类Collection、Map，Collection又分三大类List、Set、Queue队列

#### Set 
Set 与 List, Queue 的主要区别是不会有重复元素

#### ArrayList & LinkedList
* 没有加锁，线程不安全。  
* ArrayList是基于数组实现的，LinkedList是基于双链表实现的。
* LinkedList还实现了Deque接口，Deque接口是Queue接口的子接口，它代表一个双向队列，因此LinkedList可以作为双向对列。
* 因为Array是基于索引(index)的数据结构，它使用索引在数组中搜索和读取数据是很快的，可以直接返回数组中index位置的元素，因此在随机访问集合元素上有较好的性能。Array获取数据的时间复杂度是O(1),但是要插入、删除数据却是开销很大的，因为这需要移动数组中插入位置之后的的所有元素。
* 相对于ArrayList，LinkedList的随机访问集合元素时性能较差，因为需要在双向列表中找到要index的位置，再返回；但在插入，删除操作是更快的。
<!--more-->

#### Queue
* **Queue 实现的实际上是一个队列，有进有出，它实现了很多对线程友好的API offer、peek、poll，他的一个子类型叫 BlockingQueue对线程友好的API又添加了put和take，这两个实现了阻塞操作，这个是在其他的List、 Set里面都是没有的。这里面最重要的就是是叫做阻塞队列，它的实现的初衷就是为了线程池、高并发做准备的。**
* Queue里面还有一个子接口叫Deque叫双端队列，一般的队列只是从一端往里扔从另一端往外取。 Deque就是说你可以从反方向装从另外一个方向取。

| Queue Method | Equivalent Deque Method | 说明 |
| --- | --- | --- |
| add(e) | addLast(e) | 向队尾插入元素，失败则抛出异常 |
| offer(e) | offerLast(e) | 向队尾插入元素，失败则返回false |
| remove(e) | removeFirst(e) | 获取并删除首元素，失败则抛出异常 |
| poll(e) | pollFirst(e) | 获取并删除首元素，失败则返回null |
| element(e) | getFirst(e) | 获取但不删除首元素，失败则抛出异常 |
| peek(e) | peekFirst(e) | 获取但不删除首元素，失败则返回null |

#### Vector & HashTable
最开始java1.0容器里只有两个，第一个叫Vector可以单独的往里扔，还有一个是Hashtable是可以一对一对往里扔的。Vector相对于实现了List接口，Hashtable实现了Map接口，Vector 和 Hashtable 自带锁所以性能低。

#### HashMap
HashMap没有锁，线程不安全,他虽然速度比较快，但是多线程时数据会出问题。

##### Collections.synchronizedHashMap
Map<String,String> map = Collections.synchronizedMap(new HashMap<String,String>());  
用的是SynchronizedMap这个方法，给HashMap我们手动加锁，它的源码自己做了一个Object，然后每次都是SynchronizedObject，严格来讲他和那个Hashtable效率上区别不大。

#### ConcurrentMap 接口
##### ConcurrentHashMap & ConcurrentSkipListMap
ConcurrentHashMap是多线程里面真正用的，提高效率主要提高在读上面，由于它往里插的时候内部又做了各种各样的判断，本来是链表的，到8之后又变成了红黑树，然后里面又做了各种各样的 CAS 的判断，所以他往里插的速度是要更低一些的。  
ConcurrentSkipListMap 通过**跳表**来实现的高并发容器并且这个Map是有排序的;  
这两个的区别一个是有序的一个是无序的，同时都支持并发的操作
  
#### CopyOnWrite 
###### CopyOnWriteArrayList & CopyOnWriteArraySet & CopyOnWriteMap
写时复制，当Write的时候我们要进行复制。这个原码非常简单，当我们需要往里面加元素的时候，把里面的元素得复制出来，再添加一个位置存放新元素。 而且在写的时个有加锁，但在读的时候没有锁。在写的时候特别少，读的时候很多的情况下，在这个时候就可以考虑CopyOnWrite这种方式来提高效率。
```
//CopyOnWriteMap.put方法
public V put(K key, V value) {
    synchronized(this) {
        Map<K, V> newMap = new HashMap(this.internalMap);
        V val = newMap.put(key, value);
        this.internalMap = newMap;
        return val;
    }
}

//CopyOnWriteArrayList.add
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}

//CopyOnWriteArraySet 中实际是由CopyOnWriteArrayList存放的，在add的时候直接调用的CopyOnWriteArrayList.addIfAbsent(...)
```

#### BlockingQueue 接口
BlockingQueue的概念重点是在Blocking上，Blocking阻塞，Queue队列，是阻塞队列。  
BlockingQueue在 Queue的基础上又添加了两个方法，这两个方法一个叫put，一个叫take。这两个方法是真真正正的实现了阻塞。put往里装如果满了的话我这个线程会阻塞住，take往外取如果空了的话线程会阻塞住。所 以这个BlockingQueue就实现了生产者消费者里面的那个容器。

##### LinkedBlockingQueue
用链表实现的BlockingQueue，是一个无界队列

##### ArrayBlockingqueue
ArrayBlockingQueue是有界的，可以指定它一个固定的值10，它容器就是10，那么当你往里面扔容器的时候，一旦他满了这个put方法就会阻塞住。然后你可以看看用add方法满了之后他会报异常。 offer用返回值来判断到底加没加成功，offer还有另外一个写法你可以指定一个时间尝试着往里面加1秒钟，1秒钟之后如果加不进去它就返回了.

##### DelayQueue
DelayQueue可以实现在时间上的排序，这个DelayQueue能实现按照在里面等待的时间来进行排序。

##### SynchronousQueue
SynchronousQueue容量为0，就是这个东西它不是用来装内容的，SynchronousQueue是专门用来两 个线程之间传内容的，给线程下达任务。 
这个Queue和其他的很重要的区别就是 你不能往里头装东西，只能用来阻塞式的put调用，要求是前面得有人等着拿这个东西的时候你才可以 往里装，但容量为0，其实说白了就是我要递到另外一个的手里才可以。

##### TransferQueue   
TransferQueue传递，实际上是前面这各种各样Queue的一个组合，它可以给线程来传递任务，以此同时不像是SynchronousQueue只能传递一个，TransferQueue做成列表可以传好多个。比较牛X的是它添加了一个方法叫transfer，如果我们用put就相当于一个线程来了往里一装它就走了。transfer就是装完在这等着，阻塞等有人把它取走我这个线程才回去干我自己的事情。  
一般使用场景:是我做了一件事情，我这个事情要求有一个结果，有了这个结果之后我可以继续进行我下面的这个事情的时候，比方说 我付了钱，这个订单我付账完成了，但是我一直要等这个付账的结果完成才可以给客户反馈。  

#### PriorityQueue
PriorityQueue特点是它内部你往里装的时候并不是按顺序往里装的，而是内部进行了一个排序。

