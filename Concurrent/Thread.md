

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
 





