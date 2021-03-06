
　　
## 阻塞队列  （BlockingQueue）    
    
在前面几篇文章中，我们讨论了同步容器（Hashtable 、 Vector）,也讨论了并发容器（ConcurrentHashMap、CopyOnWriteArrayList),
这些工具都为我们编写多线程程序提供了很大的方便。

今天我们来讨论另一种容器：阻塞队列

在前面我们接触的队列都是非阻塞队列，比如PriorityQueue、LinkedList(LinkList是双向队列，它实现了Dequeue接口)。

使用非阻塞队列的时候有一个很大的问题就是：它不会对当前的线程产生阻塞，那么在面对类似消费-生产者的模型时，就必须额外地实现同步策略
以及线程间唤醒策略，这个实现起来就非常麻烦。但是有了阻塞队列就不一样了，它会对当前线程产生阻塞队列中有了元素。当队列中有元素后，被
阻塞的线程会自动唤醒（不需要我们代码去唤醒）。这样提供了极大的方便性。

本文先讲述一下 Java.util.concurrent包下提供的主要的几种阻塞队列，然后分析了阻塞队列和非阻塞队列中的各个方法，接着分析了阻塞队列的实现
原理，最后给出了一个实际例子和几个使用场景。

一、几种主要的阻塞队列
二、阻塞队列中的方法 VS 非阻塞队列中的方法
三、阻塞队列的实现原理
四、实例和使用场景

### 一 几种主要的阻塞队列
自从Java 1.5 之后， 在Java.util.concurrent 包下提供了若干个阻塞队列，主要有以下几个：
ArrayBlockingQueue: 基于数组实现了一个阻塞队列，在创建ArrayBlockingQueue对象是必须制定容量的大小。并且可以制定公平性与非公平性，默认情况下
                    最优先能够访问队列。
LinkedBlockingQueue: 基于链表实现的一个阻塞队列，在创建LinkedBlockingQueue对象时如果不指定容量的大小，则默认大小为 Integer.MAX_VALUE.
PriorityBlockingQueue: 以上两种队列都是先进先出队列，而PriorityBlockingQueue却不是，它会按照元素的优先级对元素进行排序，按照优先级书序出队，
                        每次出兑的元素都是优先级最高的元素。注意，此阻塞队列为无界阻塞队列，即容量没有上限（通过源码就可以知道，
                        他没有容器满的标志）,前两种都是有界队列。
                        
DelayQueue: 基于 PriorityQueure,一种延时阻塞队列，DelayQueue中的元素只有当其指定的延迟时间到了，才能从队列中获取该元素。
            DelayQueue也是一个无界队列，因此往队列中插入数据的操作（生产者）永远不会阻塞，而只有获取数据的操作（消费者）才会被阻塞。
            
### 二、阻塞队列中的方法 VS 非阻塞队列中的方法

1. 非阻塞队列中的几个主要方法：
    add(E e): 将元素e插入到队列末尾，如果插入成功，则返回true,如果插入失败（即队列已经满）,则跑出异常；；
    remove(); 移除队首元素，若移除成功，则返回true,如果移除失败(队列为空), 则会跑出异常；
    offer(E e): 将元素e 出入到队列末尾，如果插入成功，则返回true,如果插入失败（即队列已经满）,则返回false;
    poll(): 移除并获取队首元素，若成功，则返回队首元素，否则返回 null;
    peek(): 获取队首元素，若成功返回队首元素，否则返回null

　对于非阻塞队列，一般情况下建议使用offer、poll和peek三个方法，不建议使用add和remove方法。
　因为使用offer、poll和peek三个方法可以通过返回值判断操作成功与否，而使用add和remove方法却不能达到这样的效果。
　注意，非阻塞队列中的方法都没有进行同步措施。
　
2. 阻塞队列中的几个主要方法：
    阻塞队列包括了非阻塞队列中的大部分方法，上面列举的5个方法在阻塞队列中都存在，但是要注意这5个方法在阻塞队列中都进行了同步措施。
    除此之外，阻塞队列提供了另外4个非常有用的方法。

    put(E e);
    take 
    offer(E e,long timeout,TimeUnit unit) 
    poll(long timeout, TimeUnit unit)

    put 方法用来向队尾存入元素，如果队列满，则等待。
    take 方法用来从队首取元素，如果队列为空，则等待。
    offer 方法用来向队尾存入元素，如果队列满，则等待一定时间，当时间期限达到时，如果还没有插入成功，则返回false,否则返回true
    poll 方法用来从队首取元素，如果队列空，则等待一定的时间，当时间期限达到时，如果取到，则返回null，否则返回取得的元素。
        
### 三 阻塞队列的实现原理

　前面谈到了非阻塞队列和阻塞队列中常用的方法，下面来探讨阻塞队列的实现原理，本文以ArrayBlockingQueue为例，
　其他阻塞队列实现原理可能和ArrayBlockingQueue有一些差别，但是大体思路应该类似，有兴趣的朋友可自行查看其他阻塞队列的实现源码。

首先看一下ArrayBlockingQueue类中的几个成员变量：
```
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {

private static final long serialVersionUID = -817911632652898426L;

/* The queued items  */
private final E[] items;

/* items index for next take, poll or remove */
private int takeIndex;

/* items index for next put, offer, or add. */
private int putIndex;

/* Number of items in the queue */
private int count;

/*
* Concurrency control uses the classic two-condition algorithm
* found in any textbook.
*/

/** Main lock guarding all access */
private final ReentrantLock lock;

/** Condition for waiting takes */
private final Condition notEmpty;

/** Condition for waiting puts */
private final Condition notFull;

}

    可以看出，ArrayBlockingQueue中用来存储元素的实际上是一个数组，takeIndex和putIndex分别表示队首元素和队尾元素的下标，
    count表示队列中元素的个数。
    
    lock是一个可重入锁，notEmpty和notFull是等待条件。

    下面看一下ArrayBlockingQueue的构造器，构造器有三个重载版本：
    
public ArrayBlockingQueue(int capacity) {
}
public ArrayBlockingQueue(int capacity, boolean fair) {

}
public ArrayBlockingQueue(int capacity, boolean fair,
                          Collection<? extends E> c) {
}
 　　第一个构造器只有一个参数用来指定容量，第二个构造器可以指定容量和公平性，
 　　第三个构造器可以指定容量、公平性以及用另外一个集合进行初始化。

　　然后看它的两个关键方法的实现：put()和take()：

public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    final E[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        try {
            while (count == items.length)
                notFull.await();
        } catch (InterruptedException ie) {
            notFull.signal(); // propagate to non-interrupted thread
            throw ie;
        }
        insert(e);
    } finally {
        lock.unlock();
    }
}
 　　从put方法的实现可以看出，它先获取了锁，并且获取的是可中断锁，然后判断当前元素个数是否等于数组的长度，
 　　如果相等，则调用notFull.await()进行等待，如果捕获到中断异常，则唤醒线程并抛出异常。

　　当被其他线程唤醒时，通过insert(e)方法插入元素，最后解锁。

　　我们看一下insert方法的实现：

private void insert(E x) {
    items[putIndex] = x;
    putIndex = inc(putIndex);
    ++count;
    notEmpty.signal();
}
 　　它是一个private方法，插入成功后，通过notEmpty唤醒正在等待取元素的线程。

　　下面是take()方法的实现：

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        try {
            while (count == 0)
                notEmpty.await();
        } catch (InterruptedException ie) {
            notEmpty.signal(); // propagate to non-interrupted thread
            throw ie;
        }
        E x = extract();
        return x;
    } finally {
        lock.unlock();
    }
}
 　　
 　　跟put方法实现很类似，只不过put方法等待的是notFull信号，而take方法等待的是notEmpty信号。
 　　在take方法中，如果可以取元素，则通过extract方法取得元素，下面是extract方法的实现：

private E extract() {
    final E[] items = this.items;
    E x = items[takeIndex];
    items[takeIndex] = null;
    takeIndex = inc(takeIndex);
    --count;
    notFull.signal();
    return x;
}    

```
跟insert方法也很类似。

其实从这里大家应该明白了阻塞队列的实现原理，事实它和我们用Object.wait()、Object.notify()和非阻塞队列实现生产者-消费者的思路类似，
只不过它把这些工作一起集成到了阻塞队列中实现。


 ### 示例 和 使用 场景

```
package javabasic.collection.concurrent;

import java.util.PriorityQueue;
import java.util.Queue;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

/**
 * @ClassName: BlcokedQueueTest
 * @Description:
 * 
 * 一、先使用Object.wait()和Object.notify()、非阻塞队列实现生产者-消费者模式
 * 
  * 　有没有发现，使用阻塞队列代码要简单得多，不需要再单独考虑同步和线程间通信的问题。
  * 
 *　在并发编程中，一般推荐使用阻塞队列，这样实现可以尽量地避免程序出现意外的错误。
 *
 *　阻塞队列使用最经典的场景就是  socket客户端数据  的读取和解析，读取数据的线程不断将数据放入队列，
 *    然后解析线程不断从队列取数据解析。还有其他类似的场景，只要符合生产者-消费者模型的都可以使用阻塞队列。
 * 
 * @author ruihongsun
 * @date 2016年5月26日 上午9:50:48 
 */
public class BlcokedQueueTest {
	
	public static void main(String[] args) {
		
		/**
		 * 使用 Object.wait()和Object.notify()、非阻塞队列实现生产者-消费者模式：
		 */
//		producerConsumerWaitNotify();

		/**
		 * 使用阻塞队列实现的生产者-消费者模式
		 */
		producerConsumerBlockedQueue();
		
	}
	
	/**
	 * 2. 使用阻塞队列实现的生产者-消费者模式
	 */
	private static void producerConsumerBlockedQueue() {
		int queueSize = 15;
		BlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(queueSize);
		
		ProducerBlockedQueue producerBlockedQueue = new ProducerBlockedQueue(blockingQueue, queueSize);
		ConsumerBlockedQueue consumerBlockedQueue = new ConsumerBlockedQueue(blockingQueue);
		
		producerBlockedQueue.start();
		consumerBlockedQueue.start();
	}

	/**
	 * 1. 使用 Object.wait()和Object.notify()、非阻塞队列实现生产者-消费者模式
	 */
	private static void producerConsumerWaitNotify() {
		int queueSize = 10;
		Queue<Integer> queue = new PriorityQueue<Integer>(queueSize);
		
		ProducerWaitNotify producerWaitNotify = new ProducerWaitNotify(queue, queueSize);
		ConsumerWaitNotify consumerWaitNotify = new ConsumerWaitNotify(queue);
		
		producerWaitNotify.start();
		consumerWaitNotify.start();
	}

	
}

class ConsumerWaitNotify extends Thread {

	Queue<Integer> queue = null;

	public ConsumerWaitNotify(Queue<Integer> queue) {
		this.queue = queue;
	}

	@Override
	public void run() {
		consume();
	}

	private void consume() {
		while (true) {
			synchronized (queue) {
				while (queue.size() == 0) {
					try {
						System.out.println("队列空，等待数据");
						queue.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
						queue.notify();
					}
				}
				queue.poll(); // 每次移走队首元素
				queue.notify();
				System.out.println("从队列取走一个元素，队列剩余" + queue.size() + "个元素");
			}
		}
	}
}

/* =============================    wait and notify END  ========================================  */

class ProducerWaitNotify extends Thread {
	
	private Queue<Integer> queue = null;
	private int queueSize = 0;
	
	public ProducerWaitNotify(Queue<Integer> queue, int queueSize) {
		this.queue = queue;
		this.queueSize = queueSize;
	}
	
	@Override
	public void run() {
		super.run();
		producer();
	}

	private void producer() {
		while (true) {
			synchronized (queue) {
				while (queue.size() == queueSize) {
					try {
						System.out.println("队列已经满了，等待有空余空间");
						queue.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
						queue.notify();
					}
				}
				queue.offer(1);
				queue.notify();
				System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
			}
		}
	}
	
}

class ConsumerBlockedQueue extends Thread {
	
	BlockingQueue<Integer> blockingQueue = null;
	
	public ConsumerBlockedQueue(BlockingQueue<Integer> blockingQueue) {
		this.blockingQueue = blockingQueue;
	}
	
	@Override
	public void run() {
		super.run();
		consumer();
	}

	private void consumer() {
		while (true) {
			try {
				blockingQueue.take();
				System.out.println("从队列取走一个元素，队列剩余：" + blockingQueue.size() + "个元素!");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}

class ProducerBlockedQueue extends Thread {
	
	private int queueSize = 0;
	BlockingQueue<Integer> blockingQueue = null;
	
	public ProducerBlockedQueue(BlockingQueue<Integer> blockingQueue, int queueSize) {
		this.blockingQueue = blockingQueue;
		this.queueSize = queueSize;
	}
	
	@Override
	public void run() {
		super.run();
		producer();
	}

	private void producer() {
		while (true) {
			try {
				blockingQueue.put(1);
				System.out.println("向队列中的元素插入一个元素，队列剩余空间 ：" + (queueSize - blockingQueue.size()) + "个元素");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
}

```
   
    
    
    
    
    
    
    

























