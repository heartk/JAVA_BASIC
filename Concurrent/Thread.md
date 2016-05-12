

## Thread


## Sleep

### SleepTest Example ###

```
/**
 * @ClassName: SleepThread
 * @Description:
 * 
 * sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。

 *　但是有一点要非常注意，sleep方法不会释放锁，
 *  也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。
 * 
 * @author ruihongsun
 * @date 2016年5月12日 上午9:29:54 
 */
public class SleepThreadTest {
	
	private int i = 0;

	private Object object = new Object();
	
	public static void main(String[] args) {
		SleepThreadTest sleepThreadTest = new SleepThreadTest();
		MyThread myThread1 = sleepThreadTest.new MyThread();
		MyThread myThread2 = sleepThreadTest.new MyThread();
		
		myThread1.start(); 
		// thread1 执行后 进入sleep,sleep方法不会释放锁，线程进入阻塞状态， 所以thread2 无法立即执行对象
		// 线程进入阻塞状态
		
		myThread2.start();
	}
	
	
	class MyThread extends Thread {
		
		@Override
		public void run() {
			super.run();
			synchronized (object) {
				i ++;
				System.out.println("i:" + i);
				
				try {
					System.out.println("线程" + Thread.currentThread().getName() + "进入睡眠状态！");
					Thread.sleep(10000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("线程" + Thread.currentThread().getName() + "睡眠结束！");
				i ++;
				System.out.println("i:" + i);
			}
		}
		
	}
}

```

## yield 

```
　调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。
　但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

　注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。
public static native void yield();
```
#### comment ####
A hint to scheduler that the current thread is willing to yield its current use of a processor.
The scheduler is free to ignore this hint.

Yield is heuristic attempt to improve relative progression between threads that would otherwise over-utilise a CPU.
Its use should be combined with detailed profiling and benchmarking to ensure that it actualluy has the desired effect.

It is rarely appropriate to use this method.
It may be userful for debugging or testing purposes, where it may help to reproduce bugs due to race conditions.
It may also be useful when designing concurrency control constructs such as the ones in the 
 {@link java.util.concurrent.locks} package.
 

## Join 

```
/**
 * @ClassName: JoinTest
 * @Description:
 * @author ruihongsun
 * @date 2016年5月12日 下午5:22:30 
 */
public class JoinTest {

	public static void main(String[] args) {
		System.out.println("进入线程：" + Thread.currentThread().getName());
		JoinTest joinTest = new JoinTest();
		MyThread myThread = joinTest.new MyThread();
		myThread.start();
		try {
			System.out.println("join使main等待：" + Thread.currentThread().getName());
			
			//　可以看出，当调用thread1.join()方法后，main线程会进入等待，然后等待thread1执行完之后再继续执行。
			//  实际上调用join方法是调用了Object的wait方法，这个可以通过查看源码得知：
			//----------------------
			myThread.join();
			//----------------------
			
			System.out.println("线程" + Thread.currentThread().getName() + "继续执行");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	class MyThread extends Thread {
		@Override
		public void run() {
			super.run();
			System.out.println("进入线程内部：" + Thread.currentThread().getName());
			try {
				//----------------------
				Thread.sleep(5000);
				//----------------------
			} catch (Exception e) {
				// TODO: handle exception
			}
			System.out.println("执行完毕：" + Thread.currentThread().getName());
		}
	}
	
}

```

## Interrupt

```
/**
 * @ClassName: InterruptTest
 * @Description: interrupt，顾名思义，即中断的意思。
 * 单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；
 * 另外，通过interrupt方法和isInterrupted()方法来停止正在运行的线程。
 * 
 *　以下是关系到线程属性的几个方法：

 *　1）getId
 *　用来得到线程ID

 *　2）getName和setName
 *　用来得到或者设置线程名称。

 *　3）getPriority和setPriority
 *　用来获取和设置线程优先级。

 *　4）setDaemon和isDaemon
 *　用来设置线程是否成为守护线程和判断线程是否是守护线程。

 *　守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

 *　Thread类有一个比较常用的静态方法currentThread()用来获取当前线程。
 * 
 * @author ruihongsun
 * @date 2016年5月12日 下午5:51:18
 */
public class InterruptTest {
	
	public static void main(String[] args) {
		// 中断阻塞状态的线程
		interruptBlocked();
		
		// 中断 非阻塞 状态的线程
		long currentTimeMillis1 = System.currentTimeMillis();
		interruptUnblock();
		long currentTimeMillis2 = System.currentTimeMillis();
		System.out.println(currentTimeMillis2 - currentTimeMillis1);
	}
	
	/**
	 * 用来中断一个正处于 非阻塞状态 的线程
	 * 
	 * 所以说直接调用interrupt方法不能中断正在运行中的线程。
	 */
	private static void interruptUnblock() {
		System.out.println("Integer中的最大值" + Integer.MAX_VALUE);
		UnBlockThread unBlockThread = new InterruptTest().new UnBlockThread();
		unBlockThread.start();
		try {
			Thread.sleep(2000);
		} catch (Exception e) {
			
		}
		UnBlockThread.interrupted();
	}

	/**
	 * 用来中断一个正处于阻塞状态的线程
	 */
	private static void interruptBlocked() {
		InterruptTest interruptTest = new InterruptTest();
		MyThread myThread = interruptTest.new MyThread();
		myThread.start();
		try {
			Thread.sleep(2000);
		} catch (Exception e) {
			// TODO: handle exception
		}
		myThread.interrupt();
	}

	class MyThread extends Thread {
		@Override
		public void run() {
			super.run();
			try {
				System.out.println("进入睡眠状态");
                Thread.sleep(10000);
                System.out.println("睡眠完毕");
			} catch (InterruptedException e) {
				System.out.println("得到中断的线程！");
			}
			System.out.println("run方法结束");
		}
	}
	
	class UnBlockThread extends Thread {
		@Override
		public void run() {
			super.run();
			int i = 0;
			while (i < Integer.MAX_VALUE && this.isInterrupted()) {
				System.out.println(i + "while 循环");
				i++;
			}
		}
	}

}

```



