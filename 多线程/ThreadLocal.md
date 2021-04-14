## 一、什么是ThreadLocal

从用法的角度、用通俗的话讲。一个`ThreadLocal`类的实例对象，就是一个容器，这个容器类似于[良心壶](https://baike.baidu.com/item/%E8%89%AF%E5%BF%83%E5%A3%B6)，只不过这个容器有很多的空腔，不同的人(线程)在使用这个容器时，用的是不同的空腔，所有不同人(线程)之间，是相互不干扰的。

所有也很容易明白`ThreadLocal`跟List、HashMap这些集合类一样，都是用于存放数据的。

__简单示例：__

```java
//创建一个ThreadLocal对象的实例对象，用ThreadLocal中的静态方法创建，并且初始化一个参数
static ThreadLocal<Integer> local=ThreadLocal.withInitial(()->1);
    public static void main(String[] args) {
        //循环创建10个线程
        for (int i=0;i<10;i++){
           new Thread(()->{
               //从实例对象中获取数据
               Integer in = local.get();
               //修改数据
               in=in+1;
               //将修改后的数据重新写入
               local.set(in);
               //答应输出
               System.out.println(Thread.currentThread().getName()+"=="+local.get());
           },"thread_"+i).start();
       }
    }
```

__运行结果：__每个线程执行时，都将取出的数据进行了+1操作，但结果表面，各个线程的操作都没有对其他线程产生影响，说明ThreadLocal中的数据是线程隔离的。

```txt
thread_0==2
thread_1==2
thread_3==2
thread_2==2
thread_5==2
thread_4==2
thread_7==2
thread_6==2
thread_8==2
thread_9==2
```

## 二、ThreadLocal原理

#### 2.1 例子

>有一家保险公司，给客户提供保存物品的服务，客户将要保存的物品给公司后，公司会把这个物品装进一个箱子，然后将箱子给客户，由客户带走。
>
>但是，如果客户想要往箱子里面取东西或者存东西，必须到保险公司去，由保险公司操作。
>
>然后客户还有另一个东西要保存，它去了另一家保险公司，这家保险公司看见客户有一个箱子了，就不给客户箱子了，因为这个箱子的规格是行业统一的。然后它将客户的要保存的物品放进箱子里面，就把箱子还给客户了。
>
>__注意：__虽然客户只有一个箱子，不同的公司都往往这个箱子里放，但是一家公司是不能对另一家公司存放的物品进行操作。可以理解为，这个箱子里面，有很多的格子，一家公司，只能对其中的一个格子进行操作。 
>
>一家保险公司：一个ThreadLocal类的实例对象
>
>一个客户 ：一个线程
>
>同时我们注意到，在__简单示例__代码中，我们创建一个ThreadLocal的实例对象，是通过`withInitial()`这样一个静态方法，在创建时就初始化了一个参数。这个参数就类似一个公司的名片，它会在每个顾客的箱子中，放一张。客户可以对这张名片做任何修改，都是不影响其他客户的。
>
>当然，我们也可以通过构造方法创建实例对象，就相当于该公司没有印自己的名片，客户得到的箱子就是空的，必须由客户往里面放东西

```java
//通过构造方法实例化对象，没有初始化数据。
static ThreadLocal<Integer> local=new ThreadLocal<>();
//通过ThreadLocal类的今天方法实例化对象，有初始化参数。
static ThreadLocal<Integer> local=ThreadLocal.withInitial(()->1);
```

#### 2.2 源码分析

##### 2.3.1 那个存放物品的箱子

我们可以看到，在`Thread`类中，有一个类型为`ThreadLocal.ThreadLocalMap`名为`threadLocals`的属性，这就是公司给客户发的箱子。但是可以看到目前还是`null`值，但是当线程第一次使用时，ThreadLocal会对其进行实例化操作（后续在讲）。

```java
public class Thread implements Runnable {
    
    .....
        
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
    ....
}
```

而`ThreadLocal.ThreadLocalMap`是`ThreadLocal`的一个内部类。这就是这行业统一的箱子规格。

在该内部类箱子里面，可以看到有于一个`Entry`类的数组，这个数组就相当于这个盒子里面的很多格子。一个`Entry`实例就是一个格子，这个格子就是真正存放数据的地方。

不同公司，只能使用一个格子。也就是不同的TheadLocal类的实例对象，就对应不同的数组下标。

```java
public class ThreadLocal<T> {

    .......
    
    //ThreadLocalmap类，箱子规格。
    static class ThreadLocalMap {

        //箱子里面装东西的盒子
        static class Entry extends WeakReference<ThreadLocal<?>> {

            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

       //数组的初始容量是16，如果不够，会自动扩容。
        private static final int INITIAL_CAPACITY = 16;

        //一个Entry类是数组，
        private Entry[] table;

    }
}
```

##### 2.3.2 从get()方法出发

```java
    public T get() {
        //获得当前线程对象
        Thread t = Thread.currentThread();
        //从当前线程对象去获取ThreadLocal对象。（拿客户自己的箱子）
        ThreadLocalMap map = getMap(t);
        //如果客户有箱子，下一步。
        if (map != null) {
            //找到这个客户箱子中，属于自己的那一个格子。
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        //客户没有箱子，初始化一个。
        return setInitialValue();
    }
```

在上面的代码中，其他点都比较好理解，但有一个点比较有意思，需要说一下，就是`getEntry(this)`这个方法，先看一下源码。

```java
private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
```

这一步就是__公司__（ThreadLocal的一个实例对象）在该__用户__（一个线程，一个Thread类实例对象）的__箱子__（一个ThreadLocalMap类的实例对象）里面，找属于自己公司的那一个格子。我们发现，这个公司是作为一个key，去找到对应的格子（对应的数组位置）。

但是有趣的是，这里并没有使用`HashCode()`方法，而是使用了`ThreadLocal`自带的获取hash值的方法，来看一下源码。

```java
public class ThreadLocal<T> {
    /**
     *实例对象的hash值
     */
    private final int threadLocalHashCode = nextHashCode();

    /**
     * 这是ThreadLocal类的一个静态属性，并且使用的是AtomicInteger，保证了该属性是保证这个属性全局立马可见。
     */
    private static AtomicInteger nextHashCode =new AtomicInteger();

    /**
     * 这个值是（2<<32）* 黄金分割率
     */
    private static final int HASH_INCREMENT = 0x61c88647;

    /**
     *每一个ThreadLocal的新对象的threadLocalHashCode的值，都是当前nextHashCode*HASH_INCREMENT+HASH_INCREMENT
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
}
```

在这个求hash值的方法里面，用了一个`0x61c88647`值。这个方法可以归结成一个式子

```txt
nextHashCode=nextHashCode*0x61c88647+0x61c88647
```

`tnextHashCode`是`ThreadLocal`类的一个静态属性，这样就保证了，每一个`ThreadLocal`对象的`threadLocalHashCode`值是唯一的。

同时在上一步中，通过ThreadLocal的哈希值去找对应的数组下标时（找到箱子中，属于自己的格子）用了一个位与(`&`)操作。

```txt
int i = key.threadLocalHashCode & (table.length - 1);
Entry e = table[i];
```

而在求hash值出现的那个怪异数字的作用就是，可以使得每一个`ThreadLocal`实例的hash值，经过与数组长度的`&`操作后，能均匀的分散在数组中，绝对不会产生哈希冲突。那个怪异数字（`0x61c88647`）就是关键。这里涉及到的知识点，是`斐波那契数列`。至于为什么跟黄金分割比例挂上钩，就能很均与的分散，我也说不清楚，今后如果有时间，再去研究。

##### 2.3.3 从set()方法出发

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
```

