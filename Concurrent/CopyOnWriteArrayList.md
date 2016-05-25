
## 并发容器之CopyOnWriteArrayList

*[原文链接 直接点击](http://ifeve.com/java-copy-on-write/)

    Copy-On-Write简称 COW , 是一种用于程序设计中的优化策略，从一开始大家都在共享同一个内容，当某个人要修改这个内容的
    时候，才会真正把内容Copy出去 形成一个新的内容然后再改，这是一种延时懒惰策略。
    
    从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器，它们是 CopyOnWriteArrayList和CopyOnWriteArraySet.
    CopyOnWrite容器非常有用，可以在非常多的场景中使用到。

### 什么是 CopyOnWrite 容器

CopyOnWrite容器即写复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将容器
进行Copy,复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再讲原容器的引用指向新的容器。
这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。
所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

### CopyOnWriteArrayList的实现原理

    在使用CopyOnWriteArrayList 之前，我们先阅读其源码来了解下它是如何实现的。
    以下代码是向CopyOnWriteArrayList中add方法的实现（向CopyOnWriteArrayList里添加元素），可以发现在添加
    的时候是需要加锁的，否则多线程写的时候会Copy出N个副本出来。
    
```
/**
 * Appends the specified elements to the end of this list.
 * 
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 *
 */
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();

    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }

}
```
读的时候不需要加锁，如果读的时候多个线程正在向 CopyOnWriteArrayList添加数据，
读还是会读到旧的数据，因为写的时候不会锁住CopyOnWriteArrayList。

```
public E get(int index) {
    return get(getArray(), index);
}
```
JDK 中并没有提供 CopyOnWriteMap ,我们可以参考 CopyOnWriteArrayList

```
import java.util.Collection;
import java.util.Map;
import java.util.Set;

public class CopyOnWriteMap<K, V> implements Map<K, V>, Callable {
    private volatile Map<K, V> internalMap;
    
    public CopyOnWriteMap() {
        internalMap = new HashMap<K, V>();
    }
    
    public V put(K key, V value) {
        synchronized(this) {
            Map<K, V> newMap = new HashMap<K, V>(internalMap);    
            internalMap = newMap;
            return val;
        }
    }
    
    public V get(Object key) {
        return internalMap.get(key);
    }
    
    public void putAll(Map<? extends K,? extends V> newData) {
        synchronized (this) {
         Map<K, V> newMap = new HashMap<K, V>(internalMap);
         newMap.putAll(newData);
         internalMap = newMap;
        }
    }
}
```
实现很简单，只要了解 CopyOnWrite 机制，我们可以实现各种 CopyOnWrite容器，并且在不同的应用场景中使用。




## CopyOnWtirte的应用场景
CopyOnWrite 并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，
假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。
这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。当用户搜索是，会检查当前关键字在不在
黑名单当中，如果在，则提示不能搜索。实现代码如下；

```
package com.ifeve.book;
import java.util.Map;
import com.ifeve.book.forkjoin.CopyOnWriteMap;
/**
 * 黑名单服务
 * @author fangtengfei
 */
public class BlackListServiceImpl {
    private static CopyOnWriteMap<String, Boolean> blackListMap = new CopyOnWriteMap<String, Boolean>(); 

    public static boolean isBlackList(String id) {
        return blackListMap.get(id) == null? false : true;
    }
    
    public static void addBlack(String id) { 
        blackListMap.put(id, Boolean.TRUE); 
    } 
    
    /**
     *  批量添加黑名单
     */
     public static void addBlackList(Map<String, Boolean> ids) {
        blackListMap.putAll(ids);
     }    
     
}

```                 
代码很简单，但是使用CopyOnWriteMap需要注意两件事：
1. 减少扩容开销，根据实际需要，初始化CopyOnWriteMap的大小，避免写时CopyOnWriteMap的开销。
2. 使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数，
    如使用上面代码里的addBlackList方法

### CopyOnWrite的缺点
CopyOnWrite容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题。所以在开发的时候要注意下。

    *内存占用问题*
    因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，
    旧的对象和新写入的对象，(注意复制的时候只是复制容器里的引用，只是在写的时候会创建新的对象添加到新的容器里，
    而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入
    100M数据进去，内存就会占用300M,那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务忧郁每晚使用
    CopyOnWrite机制更新大对象，造成了每晚15秒的Full GC ,应用响应时间也随之变长。
    
    *针对内存占用问题*
    可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素是10进制的数字，可以
    考虑把它压缩成32进制 或64进制。或者不实用CopyOnWrite容器，而使用其他并发容器，如ConcurrentHashMap
    
    *数据一致性问题*
    CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的数据，马上能读到，
    请不要用CopyONWrite容器，而是用其他并发容器
    
    下面这篇文章验证了CopyOnWriteArrayList和同步容器的性能：

　　http://blog.csdn.net/wind5shy/article/details/5396887

　　下面这篇文章简单描述了CopyOnWriteArrayList的使用：

　　http://blog.csdn.net/imzoer/article/details/9751591
　　









































