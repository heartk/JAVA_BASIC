
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









































