
## Java并发编程：CountDownLatch、CyclicBarrier和Semaphore

Java并发编程：CountDownLatch、CyclicBarrier和Semaphore

　　在java 1.5中，提供了一些非常有用的辅助类来帮助我们进行并发编程，比如CountDownLatch，CyclicBarrier和Semaphore，今天我们就来学习一下这三个辅助类的用法。

　　以下是本文目录大纲：

　　一.CountDownLatch用法

　　二.CyclicBarrier用法

　　三.Semaphore用法

　　若有不正之处请多多谅解，并欢迎批评指正。

　　请尊重作者劳动成果，转载请标明原文链接：

　　http://www.cnblogs.com/dolphin0520/p/3920397.html
　　
###  一.CountDownLatch用法　
　CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。
　比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。
　CountDownLatch类只提供了一个构造器：
```
  public CountDownLatch(int count) {  };  //参数count为计数值
```
然后下面这3个方法是CountDownLatch类中最重要的方法：

```
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { }; 

//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  

public void countDown() { };  //将count值减1
```
下面看一个例子大家就清楚CountDownLatch的用法了：

```
package javabasic.multiThread.assistclass;

import java.util.concurrent.CountDownLatch;

/**
 * @ClassName: CountDownLatchTest
 * @Description: CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。
 * @author ruihongsun
 * @date 2016年6月3日 下午4:18:47 
 */
public class CountDownLatchTest {

	public static void main(String[] args) {
		executeCountDownLatch();
	}
	
	private static void executeCountDownLatch() {
		final CountDownLatch latch = new CountDownLatch(2);
		new Thread() {
			@Override
			public void run() {
				try {
					super.run();
					System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
					Thread.sleep(3000);
					System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
					latch.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}.start();
		
		new Thread() {
			@Override
			public void run() {
				try {
					super.run();
					System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
					Thread.sleep(3000);
					System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
					latch.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}.start();
		
		try {
			System.out.println("等待2个子线程执行完毕...");
			latch.await();
			System.out.println("2个子线程已经执行完毕");
			System.out.println("继续执行主线程");
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
}
```
执行结果：
```
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```

### 二.CyclicBarrier用法

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。
叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。
我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：
```
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
```
　参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

　然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：
```
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit) throws InterruptedException,BrokenBarrierException,TimeoutException { };

```
第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

　　第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

　　下面举几个例子就明白了：

　　假若有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，
　　这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了：























































