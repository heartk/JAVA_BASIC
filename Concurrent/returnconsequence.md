
## Java并发编程：Callable、Future和FutureTask

　在前面的文章中我们讲述了创建线程的2种方式，一种是直接继承Thread，另外一种就是实现Runnable接口。

　　这2种方式都有一个缺陷就是：在执行完任务之后无法获取执行结果。

　　如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果，这样使用起来就比较麻烦。

　　而自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。

　　今天我们就来讨论一下Callable、Future和FutureTask三个类的使用方法。以下是本文的目录大纲：

　　一.Callable与Runnable

　　二.Future

　　三.FutureTask

　　四.使用示例

　　若有不正之处请多多谅解，并欢迎批评指正。

　　请尊重作者劳动成果，转载请标明原文链接：

　　http://www.cnblogs.com/dolphin0520/p/3949310.html
　　
　　### 一.Callable 与 Runnable 
　　
　　先说一下java.lang.Runnable吧，它是一个接口，在它里面只声明了一个run()方法：
　　
```
  public interface Runnable {
      public abstract void run();
  }

```
由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果。

Callable 位于 java.util.concurrent 包下，它也是一个接口，
在它里面也只声明了一个方法，只不过这个方法叫做call()：

```
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```
　　
　　
　　
　　
　　
