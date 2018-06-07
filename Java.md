Java 复习笔记

#线程和进程的关系？

#多线程
## 原子性、可见性、有序性
## 1.ThreadLocal
	数据隔离
## 2.volatile
## 3.CAS
## synchronized
	数据共享
	
#RxJava原理和优点？



#Java有几种引用？

 **强引用>软引用>弱引用>虚引用**
  
  > 除了强引用，其它的引用对象内存都可能被GC回收

- 强引用
  
  > 强引用可以直接访问目标对象。 强引用所指向的对象在任何时候都不会被系统回收。JVM宁愿抛出OOM异常，也不会回收强引用所指向的对象。 强引用可能导致内存泄露。  
- 软引用

  > 可以通过java.lang.ref.SoftReference使用软引用。一个持有软引用的对象，不会被JVM很快回收，JVM会根据当前堆的使用情况来判断何时回收。当堆的使用率临近阈值时，才会回收软引用的对象。
  
  > 需要注意的是，在垃圾回收器对这个Java对象回收前，SoftReference类所提供的get方法会返回Java对象的强引用，一旦垃圾线程回收该Java对象之后，get方法将返回null。所以在获取软引用对象的代码中，一定要判断是否为null，以免出现NullPointerException异常导致应用崩溃。
  
  如果只是想避免OutOfMemory异常的发生，则可以使用软引用。
- 弱引用
  
  > 弱引用是一种比软引用较弱的引用类型。在系统GC时，只要发现弱引用，不管系统堆空间是否足够，都会将对象进行回收。但是，由于垃圾回收器的线程通常优先级很低，因此，并一不定能很快的发现持有弱引用的对象。这种情况下，弱引用对象可以存在较长的一段时间。一旦一个弱引用对象被垃圾回收器回收，便会加入到一个注册引用队列中。
  
  如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用。
  
  >软引用，弱引用都非常适合来保存那些可有可无的缓存数据。如果这样做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间。
- 虚引用

 > 虚引用是所有引用类型中最弱的一个。一个持有虚引用的对象，和没有引用几乎是一样的，随时都可能被垃圾回收器回收。当试图通过虚引用的get()方法取得强引用时，总是会失败。并且，虚引用必须和引用队列一起使用，它的作用在于跟踪垃圾回收过程。 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，销毁这个对象，奖这个虚引用加入引用队列。
 
 
 |引用类型|被垃圾回收时间|用途|生存时间|
 |-------------|------------|-----------|---------------|
 |强引用|从来不会|对象的一般状态|JVM停止时|
 |软引用|内存不足|对象缓存|内存不足时|
 |弱引用|GC垃圾回收时总是|对象缓存|GC运行后释放|
 |虚引用|未知|未知|未知|
 
  
#内部类

- **成员内部类**
- **匿名内部类**
- **静态内部类**
- **局部内部类**

#接口和内部类

- 如果多个接口具有相同的方法，可以用内部类来避免重名方法的多继承，即内部类来实现接口的方法，重新暴露该方法。

#内存屏障 Memory Barrier/内存栅栏 Memory Fence

> 是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序。（也就是让一个CPU处理单元中的内存状态对其它处理单元可见的一项技术。）
> 

###1.LoadLoad屏障

对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

###2.StoreStore屏障

对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。

###3.LoadStore屏障

对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。

###4.StoreLoad屏障

对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。

>内存屏障阻碍了CPU采用优化技术来降低内存操作延迟，必须考虑因此带来的性能损失。为了达到最佳性能，最好是把要解决的问题模块化，这样处理器可以按单元执行任务，然后在任务单元的边界放上所有需要的内存屏障。采用这个方法可以让处理器不受限的执行一个任务单元。合理的内存屏障组合还有一个好处是：缓冲区在第一次被刷后开销会减少，因为再填充改缓冲区不需要额外工作了。

#Happens-before原则

#如何理解Java的跨平台特性

###1.Java跨平台的基础是JVM(Java Virtual Machine)
   
   >JVM是一个软件，它基于不同的操作系统和处理器实现了不同的版本，JVM本身是依赖于操作系统的，它仅仅是一个容器，用来解释Java字节码，翻译成不同操作系统和处理器下的机器码。
   > 最典型的例子，在C、C++中的基本数据类型被JVM统一了实现，这得益于JVM的翻译能力，它实现了不同操作系统下的具体实现。
   > JVM把操作系统移植的兼容性部分对程序员屏蔽了，我们不需要看到具体的C的各种适配操作系统和处理器的条件宏和数据类型转换，以及系统API的差异。统一由JVM重新封装了，因此可以理解为一个中间件，类似于我们的对各种第三方框架的抽象和封装，让它表面上是有统一的行为，对具体实现不可见。
   
###2. 分层

Byte Code(Java, OS 独立)  -> 一次编码，到处运行
---
JVM(C/C++实现,OS和处理器相关) -> 字节码解释成机器码
---
Binary Code(机器语言，处理器相关)
---

###3.跟JavaScript在各大浏览器中兼容性比较

> JavaScript/HTML等具有类似的特性，一次编写，跨浏览器支持。
原理类似，各浏览器自身是一个程序或者说软件，我们理解成容器，它负责将JS/HTML代码解释，然后渲染。
>
> 但是我们知道，不同的操作系统中的系统API是不同的，MacOS和Windows下的可执行程序的架构是不同的，二进制程序汇编指令也是不同的，因此跨平台的基础就是中间件屏蔽了这些差异，这个中间件就是浏览器。当我们在JS中调用canvas等api渲染界面的时候，浏览器仍然需要将这些操作解释成系统级别的API，最终调用操作系统的API去绘制图形，因为只有操作系统自身才能调用到硬件来绘制界面到真实的显示器上（中间是各种驱动）。





#JVM
- JVM内存模型
- JVM调优

#编写线程安全的单例
- Lazy
  
  ```
   /**
     * Usage
     * SingletonLazy.getInstance();
     */
    public class SingletonLazy
    {
        //必须volatile禁止指令重排，保证内存可见性。
        private static volatile SingletonLazy instance = null;

        private SingletonLazy() {}

        public  static SingletonLazy getInstance()
        {
            if (instance == null)
            {
                synchronized (SingletonLazy.class)
                {
                    //再次检查是因为多线程环境下可能多次创建对象
                    if (instance == null) //double check for multiple threads
                    {
                        instance = new SingletonLazy();//because new Singleton1() is not atomic op
                    }
                }
            }
            return instance;
        }
    }

  ```
- Eager
  
  ```
   /**
     * Usage
     * SingletonEager.getInstance();
     * 弱点是只要类加载就会初始化该单例对象，对于大对象不理想。
     */
    public class SingletonEager
    {
        //One instance always exists
        private static SingletonEager instance = new SingletonEager();
        private SingletonEager() {}
        public static SingletonEager getInstance()
        {
            return instance;
        }
    }
  ```
- Static Inner Class 

  ```
   /**
     * Usage:
     * SingletonWithStaticHolder.getInstance();
     */
    public class  SingletonWithStaticHolder
    {
        private SingletonWithStaticHolder() {}
        //延迟加载，只有引用到该静态内时才创建
        private static class SingletonHolder {
            private static SingletonWithStaticHolder instance = new SingletonWithStaticHolder();
        }
        public static SingletonWithStaticHolder getInstance()
        {
            return SingletonHolder.instance;
        }
    }
  ```
- **Enum** 推荐做法

  ```
    public static class Camera
    {

    }
  	 /**
     *  Usage:
     *  SingletonEnum.INSTANCE.getCamera();
     */
    public enum SingletonEnum {
        INSTANCE;
        private Camera camera = null;
        private SingletonEnum()
        {
            camera = new Camera();
        }

        public Camera getCamera()
        {
            return camera;
        }
    }
  ```

#CAS
三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。
##AtomicStampedReference
通过控制变量值的版本来保证CAS的正确性，解决ABA问题

#COW(Copy on write)
> 是一种用于程序设计中的优化策略。其基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器,它们是CopyOnWriteArrayList和CopyOnWriteArraySet。CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

**1. CopyOnWrite容器**

> CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行**并发的读，而不需要加锁**，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种**读写分离的思想，读和写不同的容器**。
> 

#设计模式
### 单例模式
### 简单工厂(静态工厂)
### 抽象工厂
### Builder
### Adapter
### Proxy
### Medidator
### Observer



#对象的equals等价关系

- 自反性

> 对于任何非null的引用值x,x.equals(x)必须返回true

- 对称性

> 对于任何的非null引用值x和y,当且仅当x.equals(y)返回true时,y.equals(x)必须返回true

- 传递性

> 对于任何非null的引用值x,y,z,如果x.equals(y)返回true并且y.equals(z)也返回true，那么x.equals(z)必须返回true

- 一致性 

> 对于任何非null的引用值x,y,只要equals的比较操作在对象中所用的信息没有发生改变，多次调用x.equals(y)就会一致的返回true或false。

- 非空性 

> 对于任何的非null的引用值x，x.equals(null)必须返回false
