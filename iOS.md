iOS面试复习
#runtime机制

> objective-c代码总是先预编译成C代码，runtime机制也是基于C的实现。
> 消息机制是runtime的基础。研究runtime机制基本上就是阅读和理解对应的C代码。

- objc_msgSend（C代码)

  ```
  void objc_msgSend(id self,SEL op, ...)
  
  objc_msgSend(self,@selector(doSomethingWithVar:),var1);
  ```
  
- 消息转发机制
 - clang -rewrite-objc xxx.mm
 - `+ (BOOL) resolveInstanceMethod:(SEL)aSEL`
 - `– (id)forwardingTargetForSelector:(SEL)aSelector`
 - `– (void)forwardInvocation:(NSInvocation *)anInvocation `
 - `doesNotRecognizeSelector `

- 方法对换
  
  ```
  void method_exchangeImplementations(Method m1, Method m2) 
  ```
-

#NSString为何要用copy？而不是strong？
- NSString是immutable，不可变类
- 为了防止mutable string被无意中修改, NSMutableString是NSString的子类, 因此NSString指针可以持有NSMutableString对象
- 对源头是NSMutableString的字符串，strong/retain仅仅是指针引用，增加了引用计数器，这样源头改变的时候，用这种strong/retain方式声明的变量（无论被赋值的变量是可变的还是不可变的），它也会跟着改变;而copy声明的变量，它不会跟着源头改变，它实际上是深拷贝。
- 对源头是NSString的字符串，无论是retain/strong声明的变量还是copy声明的变量，当第二次源头的字符串重新指向其它的地方的时候，它还是指向原来的最初的那个位置，也就是说其实二者都是指针引用，也就是浅拷贝。



#NSArray用strong修饰有什么问题？

如果不小心将一个可变数组赋值给它，将导致修改可变数组的元素的时候导致该“不可变数组”的元素发生变化。实际上指向了可变数组的内存。

#NSMutableArray用copy修饰有什么问题？

初始化的时候会导致得到一个不可变数组，增加修改的时候会崩溃。doesnotreconginizeselector

#循环引用
- weak解除循环引用
- block中的self并不是总是引起循环引用
  如果只是block引用了self，self并不持有block，并不会引起循环引用问题。

#Block
- 是一个NSObject对象
- ARC下默认copy到堆上

#category使用和实现机制

- 实现原理

> ###利用runtime动态修改 methodList 的值来添加成员方法，这也是 Category 实现的原理，因此它不能增加成员变量。添加数据可以通过关联对象来绑定一个对象。

- 使用场景
  - 为已有的类增加方法
  - 可以把类的实现分开在几个不同的文件里面。
  	a) 可以减少单个文件的体积 
  	b) 可以把不同的功能组织到不同的category里 
  	c) 可以由多个开发者共同完成一个类 
  	d) 可以按需加载想要的category 
  - 声明私有方法
  - 模拟多继承
  - 把framework的私有方法公开
  
- **extension和category的区别?**
- extension 
> ###extension在编译期决议，它就是类的一部分，在编译期和头文件里的@interface以及实现文件里的@implement一起形成一个完整的类，它伴随类的产生而产生，亦随之一起消亡。extension一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类比如NSString添加extension。
###extension可以添加实例变量.
- category
> ###category是在运行期决议的。
###category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）。但是通过关联对象可以绑定数据成员。

```
typedef struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
} category_t;
```
上面结构表面，category可以增加实例方法，类方法，实现协议，添加属性（关联对象）。不能增加实例变量。

**通过编译成C代码可以看的更清晰**

` clang -rewrite-objc MyClass.m `

- category注意事项
  - 1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA
  - 2)、category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法就会停止。

**+load**

- 1)、附加category到类的工作会先于+load方法的执行
- 2)、+load的执行顺序是先类，后category，而category的+load执行顺序是根据编译顺序决定的。
目前的编译顺序是这样的：

  

|代码形式|编译器/运行期|实例变量|和类的关系|类的源代码|
|-------------|----------------|---------------|---------------|-------------|
|@interface MainViewController ()<UIGestureRecognizerDelegate|编译器决定|可以增加实例变量|是类的一部分，跟类的产生而产生，随类的消亡一起消亡.用来隐藏类的私有信息。|需要类的源代码|
|@interface MainViewController (Gesture)|运行期决定|不可以增加实例变量，但是可以绑定一个数据对象|不是类的组成部分，由运行期修改类的方法数组增加的方法|不需要类的源代码|
#类方法load和initialize的区别
> ###1、+load方法当类或分类添加到object-c runtime时被调用，子类的+load方法会在它所有父类的+load方法之后执行，而分类的+load方法会在它的主类的+load方法之后执行。但不同的类之间的+load方法的调用顺序是不确定的，所以不要在此方法中用另一个类。

> ###2、+load方法不像普通方法一样，它不遵循那套继承规则。如果某个类本身没有实现+load方法，那么不管其它各级超类是否实现此方法，系统都不会调用。+load方法调用顺序是：SuperClass -->SubClass --> CategaryClass。

> ###3、+initialize是在类或者它的子类接受第一条消息前被调用，但是在它的超类接收到initialize之后。也就是说+initialize是以懒加载的方式被调用的，如果程序一直没有给某个类或它的子类发送消息，那么这个类的+initialize方法是不会被调用的。

> ###4、+initialize方法和+load方法还有个区别，就是运行期系统完整度上来讲，此时可以安全使用并调用任意类中的任意方法。而且，运行期系统也能确保+initialize方法一定会在“线程安全的环境”中执行，这就是说，只有执行+initialize的那个线程可以操作类或类实例，其他线程都要阻塞等着+initialize执行完。

> ###5、+initialize方法和其他类一样，如果某个类未实现它，而其超类实现了，那么就会运行超类的实现代码。如果本身和超类都没有实现，超类的分类实现了，就会去调用分类的initialize方法。如果本身没有实现，超类和父类的分类实现了就会去调分类的initialize方法。不管是在超类中还是分类中实现initialize方法都会被调多次，调用顺序是SuperClass -->SubClass。

## category通过关联对象增加属性的原理

> 运行时通过AssociationsManager里面是由一个静态AssociationsHashMap来存储所有的关联对象的，这是一个全局的map，key是关联对象的指针地址,value是另外一个AssociationsHashMap，里面保存了关联对象的kv对。
 
#KVC实现原理

> KVC运用了一个isa-swizzling技术。isa-swizzling就是类型混合指针机制。KVC主要通过isa-swizzling，来实现其内部查找定位的。isa指针，如其名称所指，（就是is a kind of的意思），指向维护分发表的对象的类。该分发表实际上包含了指向实现类中的方法的指针，和其它数据。

一个对象在调用setValue的时候，

-（1）首先根据方法名找到运行方法的时候所需要的环境参数。

-（2）他会从自己isa指针结合环境参数，找到具体的方法实现的接口。

-（3）再直接查找得来的具体的方法实现。



#KVO使用和实现原理

> 当观察者为一个对象的属性进行了注册，被观察对象的isa指针被修改的时候，isa指针就会指向一个中间类，而不是真实的类。所以isa指针其实不需要指向实例对象真实的类。所以我们的程序最好不要依赖于isa指针。在调用类的方法的时候，最好要明确对象实例的类名。
> 
> 只有当我们调用KVC去访问key值的时候KVO才会起作用，KVO是基于KVC实现的。
> 
> 因为KVC的实现机制，可以很容易看到某个KVC操作的Key，也很容易的跟观察者注册表中的Key进行匹对。假如访问的Key是被观察的Key，那么我们在内部就可以很容易的到观察者注册表中去找到观察者对象，而后给他发送消息。

总结一下，想使用KVO有三种方法：

1)使用了KVC

使用了KVC，如果有访问器方法，则运行时会在访问器方法中调用will/didChangeValueForKey:方法；

没用访问器方法，运行时会在setValue:forKey方法中调用will/didChangeValueForKey:方法。

2)有访问器方法

运行时会重写访问器方法调用will/didChangeValueForKey:方法。

因此，直接调用访问器方法改变属性值时，KVO也能监听到。

3)显示调用will/didChangeValueForKey:方法。

```
[self willChangeValueForKey:@"isExecuting"];
 _isExecuting = YES;
[self didChangeValueForKey:@"isExecuting"];
```

#MVVM、MVP、MVC的区别
- MVC(Model-View-Controller)
  
 ![image](http://cc.cocimg.com/api/uploads/20170710/1499668090565523.png)
   
- MVVM(Model-View-ViewModel)


 ![](https://cdn-images-1.medium.com/max/1600/1*D4zpAC_ojOm21sL9q197ZQ.png)
  
  - ViewController和Model分离
  - ViewModel承担原来ViewController的主要职责:持有Model，负责大部分的App逻辑：绑定数据，获取数据，更新数据
  - ViewController必须通过ViewModel访问Model
  - ViewController目前负责用户交互，显示数据和动画等，视同View
  - View持有ViewModel的引用，反之没有
  - ViewModel持有Model的引用，反之没有
  - MVVM方便测试
  - Model变化->ViewModel属性变化->View更新
 
 ![image](http://cc.cocimg.com/api/uploads/20170710/1499668291203817.png)
 ![image](http://cc.cocimg.com/api/uploads/20170710/1499668392288990.png)
 ![image](http://cc.cocimg.com/api/uploads/20170710/1499668424844815.png)
 
 - MVVM without Databinding with Data Controller
 ![image](http://gracelancy.com/assets/post/arch0.png)


- MVP(Model-View-Presenter)

#MVVM优点和缺点
- 优点
  - 
- 缺点
  - 学习成本高
  - 数据绑定需要增加KVO代码或引入ReactiveCocoa
  - 数据绑定使得Debug更加困难了
  - ViewModel仍然承载了大量的逻辑:业务逻辑，界面逻辑，数据存储，网络

#第三方框架
- AFNetworking 原理
- FMDB原理
- SDWebImage原理

#属性的本质
- ivar+getter+setter
- 自动合成方法

#多线程
###1.进程和线程
- 进程

>进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。

- 线程

>线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。
一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行。

- 微线程:又叫协程(Python)

>tasklet运行在伪并发中，使用channel机制进行同步数据交换。python中的greenlet提供了微线程的操作。不同于多线程，它给我们提供了一种更加轻量的异步编程模式。
>
> 协程（Coroutine）提供了不同于线程的另一种方式，它首先是串行化的。其次，在串行化的过程中，协程允许用户显式释放控制权，将控制权转移另一个过程。释放控制权之后，原过程的状态得以保留，直到控制权恢复的时候，可以继续执行下去。所以协程的控制权转移也称为“挂起”和“唤醒”。

- 阻塞：阻塞调用是指调用结果返回之前，当前线程会被挂起。函数只有在得到结果之后才会返回。
- 非阻塞：非阻塞和阻塞的概念相对应，指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回。

###2.同步、异步、并发、串行、并行

同步和异步决定了要不要开启新的线程

- 同步 synchronization

> 在当前线程中执行任务，不具备开启新线程的能力
> 
> `dispatch_sync `
> 
> 将同步任务加入串行队列，会顺序执行，一般不这样做并且在一个任务未结束时调起其它同步任务会死锁。将同步任务加入并行队列，会顺序执行，但是也没什么意义。

- 异步 asynchronization

就是允许在执行某一个任务时，函数立刻返回，但是真正要执行的任务稍后完成。
异步方法并不一定永远在新线程里面执行，反之亦然。

> 在新的线程中执行任务，具备开启新线程的能力
> 
> `dispatch_async`
> 
> 将异步任务加入串行队列，会**顺序执行**，并且不会出现死锁问题。将异步任务加入并行队列，会并行执行多个任务，这也是我们最常用的一种方式。

并发和串行决定了任务的执行方式

- 串行 serial

> 一个任务执行完毕后，再执行下一个任务
> 同步线程的实现，在同一个线程内依次执行任务。
> 
> 串行队列 
  
  - `dispatch_queue_create `
  - `dispatch_get_main_queue ` 主线程相关联的串行队列

- 并行 concurrency 同时执行

并发指的是一种现象，一种经常出现，无可避免的现象。它描述的是“多个任务同时发生，需要被处理”这一现象。它的侧重点在于“发生”。

> **其实是真正的异步，多核CPU可以同时开启多条线程供多个任务同时执行，互不干扰**
> **只有多核CPU才存在并行,多个CPU独立开启线程调度**

- 并发 concurrent 同时发生

并行指的是一种技术，一个同时处理多个任务的技术。它描述了一种能够同时处理多个任务的能力，侧重点在于“运行”。

> CPU时间片调度，同时只有一个线程在执行。
> 但是可以多个线程异步执行，从外部看起来像是同时在执行，实际上是根据优先级不断抢占CPU的时间片执行代码。高优先级的抢占的时间多并且优先执行，所以上层看起来是同时执行但有优先级的概念。
> 
> **叠加并行即多处理器后更复杂，即并行同时并发,并发未必是并行的，因为可能在单CPU上并发**

并发是一种现象，面对这一现象，我们首先创建多个线程，真正加快程序运行速度的，是并行技术。也就是让多个CPU同时工作。而多线程，是为了让多个CPU同时工作成为可能。

> 并发队列

  - `dispatch_get_global_queue ` 全局的并发队列

 
###3.GCD(Grand Central Dispatch)
- 任务(执行什么操作)
- 队列(存放任务的队列,FIFO)

> 同步函数不具备开启线程的能力，无论是什么队列都不会开启线程；
>
>  异步函数具备开启线程的能力，开启几条线程由队列决定（串行队列只会开启一条新的线程，并发队列会开启多条线程）。
> 　　
> 同步函数
　　（1）并发队列：不会开线程
　　（2）串行队列：不会开线程
　
> 异步函数
　　（1）并发队列：能开启N条线程
　　（2）串行队列：开启1条线程
　　
　　
　　同步方法不一定在本线程，异步方法方法也不一定新开线程（考虑主队列）。
　　
　　

###4. dispatch_group
把一组任务提交到队列中，这些队列可以不相关，然后监听这组任务完成的事件。

 - `dispatch_group_create`创建一个调度任务组
 - `dispatch_group_async` 把一个任务异步提交到任务组里
 - `dispatch_group_enter/dispatch_group_leave` 这种方式用在不使用`dispatch_group_async`来提交任务，且必须配合使用
 - `dispatch_group_notify` 用来监听任务组事件的执行完毕
 - `dispatch_group_wait` 设置等待时间，在等待时间结束后，如果还没有执行完任务组，则返回。返回0代表执行成功，非0则执行失败
  
> 使用`dispatch_group_wait`，可以阻塞线程，等待group的任务执行完毕，才能继续执行后续任务
>
>  使用`dispatch_group_notify`，不会阻塞线程（group外的线程执行顺序不受影响），而且可以在执行完成group的任务后进行操作
 

- 场景1:

>现在有4个任务，任务1、任务2、任务3、任务4. 任务3必须在任务2之后，任务4必须在前3个任务都执行完成后，才能执行，并且需要在主线程更新UI。

>思路分析：
任务3必须在任务2之后，所以这两个必须串行执行，同时，任务2和3整体可以和任务1并行执行，最后，任务4只能等待前3个任务全部执行完成，才能执行。这里就可以用group快速实现场景需求。

 ```
 -(void)dispatchGroup
 {
    dispatch_queue_t globalQuene = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_queue_t selfQuene = dispatch_queue_create("myQuene", 0);
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, globalQuene, ^{
        NSLog(@"run task 1");
    });
    dispatch_group_async(group, selfQuene, ^{
        NSLog(@"run task 2");
    });
    dispatch_group_async(group, selfQuene, ^{
        NSLog(@"run task 3");
    });
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"run task 4");
    });
}
 ```
 
 ```
 A)
dispatch_group_async(group, queue, ^{ 
　　// 。。。 
}); 
<==>
B) 
dispatch_group_enter(group);
dispatch_async(queue, ^{
　　//。。。
　　dispatch_group_leave(group);
});
 ```

- 场景2:
有3个异步请求任务，任务1、2、3，在3个任务全部完成之后，需要执行任务4，用以显示界面数据。

```
-(void)disGroupEnterAndLeave{
    dispatch_queue_t globalQuene = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    
    //任务1
    dispatch_group_enter(group);
    dispatch_async(globalQuene, ^{
         NSLog(@"run task 1");
        sleep(1);
        dispatch_group_leave(group);
    });
    
    //任务2
    dispatch_group_enter(group);
    dispatch_async(globalQuene, ^{
        NSLog(@"run task 2");
        sleep(2);
        dispatch_group_leave(group);
    });
    
    //任务3
    dispatch_group_enter(group);
    dispatch_async(globalQuene, ^{
        NSLog(@"run task 3");
        sleep(3);
        dispatch_group_leave(group);
    });
    
    //一直等待完成
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
  
    //任务4
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"run task 4");
    });
    
}
```

###5. NSOperationQueue

- 1.将NSOperation添加到NSOperationQueue，使其异步执行 === GCD并行异步队列。
- 2.NSOperationQueue可以设置依赖关系 ~= GCD Group notify / GCD Group Wait / GCD Barrier。
- 3.可以设置最大并发数量（同时执行的线程数） === GCD 信号量
- 4.NSOperationQueue可以优先级
- 5.NSOperationQueue可以使用KVO来监听任务完成状态
- 6.可以取消线程任务
- 7.NSOperation可以子类化，是一个类，GCD是C函数。

 
###6. NSThread

#TableView性能优化
- VVeboTableViewDemo 性能优化Demo
- **优化方法****
  - 提前计算并缓存高度
  - 自定义Cell**异步绘制**
    drawRect自身是异步绘制
  - 滑动时按需加载
  - 预加载数据
  - 正确使用reuseIdentifier来重用Cells
  - 尽量使所有的view opaque，包括Cell自身
  - 尽量少用或不用透明图层
  - 如果Cell内现实的内容来自web，使用异步加载，缓存请求结果
  - 减少subviews的数量
  - 在heightForRowAtIndexPath:中尽量不使用cellForRowAtIndexPath:，如果你需要用到它，只用一次然后缓存结果
  - 尽量少用addView给Cell动态添加View，可以初始化时就添加，然后通过hide来控制是否显示，避免反复创建

#性能和内存优化
- ARC管理内存
- 在正确的地方使用 reuseIdentifier
- 尽量把views设置为不透明
  如果你有透明的Views你应该设置它们的opaque属性为YES
- 避免过于庞大的XIB
- 不要阻塞主线程
- 在Image Views中调整图片大小
- 选择正确的Collection类型
- 打开gzip压缩
- 重用和延迟加载(lazy load) Views
- Cache, Cache, 还是Cache!
- 选择是否缓存图片
  `imageNamed`的优点是当加载时会缓存图片。同时会占用内存。
   适合图片反复重用的情况
  `imageWithContentsOfFile`仅加载图片。适合加载一个大图片而且是一次性使用的情况，不缓存，节省内存。
- 避免日期格式转换
 - 任何时候重用`NSDateFormatters`都是一个好的实践。
 - 如果你可以控制你所处理的日期格式，尽量选择Unix时间戳
- 处理内存告警
 - 在app delegate中使用`applicationDidReceiveMemoryWarning:`的方法
 - 在你的自定义UIViewController的子类(subclass)中覆盖`didReceiveMemoryWarning`
 - 注册并接收 UIApplicationDidReceiveMemoryWarningNotification的通知
- 重用大开销对象
  - NSDateFormatter
  - NSCalendar
  - NSImage

  
#内存泄漏调试
- 启用Enable Zombie Objects 僵尸对象 进行悬挂指针的检测
  
  **上线打包的时候要确保关掉，否则会影响上线**
- 应用Product -> Analysis进行内存泄露的初步检测
- 可以在xcode的build setting中打开implicit retain of ‘self’ within blocks，xcode编译器会给出警告，逐个排查警告。**检查block的循环引用**
- 在Build Settings启用Analyze During 'Build'
- 应用Leak Instrument进行内存泄露查找。
- 通过查看```dealloc```是否调用查看某个class是否泄露的问题

##内存泄漏检测工具
####Facebook iOS 内存检测三剑客(FBAllocationTracker/FBMemoryProfiler/FBRetainCycleDetector)，MSLeakHunter，MLeaksFinder，PLeakSniffer等等

- 如何判断VC是否还在内存驻留？

 > 利用ARC中weak指针指向的对象在对象释放时会自动置为nil的特性来检测VC是否在内存驻留。
 
- 在什么时机检测VC是否发生内存泄露？ 

> 通过监控UINavigationController的navigation stack，可以判断一个VC的生命周期的开始和结束。就是当VC从navigation stack移除且VC的viewDidDisappear方法执行时，可以认为一个VC的生命周期即将结束。这时候就可以创建一个指向该VC的weak指针，并初始化一个定时器对VC进行延时扫描，最后通过1中的方法判断VC是否还驻留在内存从而得出VC是否发生内存泄露的结论。

#CollectionView

#ARC内部原理

#weak实现原理

> ####Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash表，Key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。
> 为什么value是数组？因为一个对象可能被多个弱引用指针指向

- 1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
- 2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
- 3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

#weak，__unsafe_unretained, unowned 与 assign区别

- __unsafe_unretained: 不会对对象进行retain,当对象销毁时,会依然指向之前的内存空间(野指针)

- weak: 不会对对象进行retain,当对象销毁时,会自动指向nil

- assign: 实质与__unsafe_unretained等同

- unsafe_unretained也可以修饰代表简单数据类型的property，weak也不能修饰用来代表简单数据类型的property。

> ####__unsafe_unretained 与 weak 比较，使用 weak 是有代价的，因为通过上面的原理可知，__weak需要检查对象是否已经消亡，而为了知道是否已经消亡，自然也需要一些信息去跟踪对象的使用情况。也正因此，__unsafe_unretained 比 __weak快,所以当明确知道对象的生命期时，选择__unsafe_unretained 会有一些性能提升，这种性能提升是很微小的。但当很清楚的情况下，__unsafe_unretained 也是安全的，自然能快一点是一点。而当情况不确定的时候，应该优先选用 __weak 。

> unowned使用在Swift中，也会分 weak 和 unowned。unowned 的含义跟 __unsafe_unretained 差不多。假如很明确的知道对象的生命期，也可以选择 unowned。

#iOS内存管理

#Autorelease机制

> 在没有手加Autorelease Pool的情况下，Autorelease对象是在当前的runloop迭代结束时释放的，而它能够释放的原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop


#关联对象
- 为分类增加数据
- getAssociatedObject
- setAssociatedObject

#设计模式
- 观察者
 - NSNotification
 - KVO
 - Delegate
- 命令模式
 - target-action
   [target performSelector:action withObject:self];
 - 少量回调用block，大量回调用delegate
 - NSInvocation(target+selector+方法签名+参数)
 - IMP = id function(id self,SEL _cmd,...)

#常量
1.NSString * const MY_CONSTANT;
2.const NSString * myVariable;
技巧：按*分割两部分，左边是数据的类型，右边是变量或常量。
1中，左边是数据类型为NSString，右边是constant，即常量，完整的说法是指向NSString（不可变类型）数据类型的常量MY_CONSTANT。
2中，左边是数据类型为const NSString，即数据类型为常量字符串，右边是变量，完整的说法是指向常量字符串的变量myVariable.
该变量可以再次指向其它任意的常量字符串。


#开发过程中遇到最深刻的BUG是什么
1.百度推送证书更新问题
  Mac OS 刚好升级到最新版本,按官方教程制作上传的证书无法验证通过。
  最后发现是Open SSL版本兼容性问题，需要下载之前的版本重新安装。百度兼容性问题。
2.ViewController无法释放问题
  知道是循环引用导致无法释放，但是因为涉及业务代码非常多，相关Block代码都需要逐一检查。
  NSTimer、通知、Block。
  在每个ViewController的dealloc做跟踪。

#JSPatch 热更新原理
- 原理
  利用JavaScriptCore.framework将JS代码利用Objective-C runtime动态解释成OC代码，封装成NSInvocation传递。

- 优点
  1. 小 利用了系统自带的JavaScriptCore.frameowrk，因此也可以通过Apple审核
  2. 支持block
- 缺点
  1.安全风险 具备Native的能力，加密下发的JS代码或https

#HTTP 2.0相比HTTP 1.0有什么优势?
- 多路复用 Multiplexing
多路复用允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息。
- 二进制分帧
应用层(HTTP/2)和传输层(TCP or UDP)之间增加一个二进制分帧层。
改进传输性能，实现低延迟和高吞吐量」
- 首部压缩（Header Compression）
  SPDY -> Deflate
  HTTP 2-> HPack
- 服务端推送（Server Push）
  服务器可以对客户端的一个请求发送多个响应

# HTTPDNS
 提高域名解析成功率
 
- 域名防劫持
- 精确调度
- 零解析延迟
- 降低解析失败率
- 按时生效

# 移动APP网络优化
## 速度
- DNS -> HTTPDNS
- HTTP 1.0 -> keep-alive
- HTTP 2.0
	
## 弱网
- 微信实践经验
  - 复合链接，提高成功率
  - 合适的超时时间
  - 调优TCP参数，优化TCP算法
## 安全
- HTTPDNS -> 防DNS劫持
- HTTPS+TLS 1.2 
- 微信自行实现了部分 TLS 1.3

#HTTP的几种方法
##GET
> 请求指定的页面信息，并返回实体主体。
GET请求请提交的数据放置在HTTP请求协议头中，GET方法通过URL请求来传递用户的输入，GET方式的提交你需要用Request.QueryString来取得变量的值。
GET方法提交数据，可能会带来安全性的问题，数据被浏览器缓存。
GET请求有长度限制。

##POST
> 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。
POST请求可能会导致新的资源的建立和/或已有资源的修改。
POST方式提交时，你必须通过Request.Form来访问提交的内容

##PUT

> 从客户端向服务器传送的数据取代指定的文档的内容。

##DELETE

> 请求服务器删除指定的页面。
DELETE请求一般返回3种码

1. 200（OK）——删除成功，同时返回已经删除的资源。
2. 202（Accepted）——删除请求已经接受，但没有被立即执行（资源也许已经被转移到了待删除区域）。
3. 204（No Content）——删除请求已经被执行，但是没有返回资源（也许是请求删除不存在的资源造成的）。

##HEAD
 
>类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头。

##CONNECT

> HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。

##OPTIONS

> 允许客户端查看服务器的性能。

##TRACE

> 回显服务器收到的请求，主要用于测试或诊断。

#iOS内存泄漏检测和原因
[https://www.jianshu.com/p/e9d989c12ff8](https://www.jianshu.com/p/e9d989c12ff8)
### 1.静态检测
- Product->Analyze
  - MRC 需要主动release和autorelease
  - C 代码需要 malloc/free,new/delete配对
 
### 2.动态检测
- instruments

> 在Allocation中我们主要关注的是Persistent和Persistent Bytes，分别表示当前时间段，申请了但是还没释放的内存数量和大小。
记住当前这两个值，然后进入某个新页面，退出该页面，观察这两个值是否增加。需要注意的是，由于有些图片调用本身是有缓存的，如果是用SDWebImage管理，则网络图片都会缓存在内存中。因此退出页面后内存有增加是正常的，而且还有些单例的内存也是不会释放的，我们可以再次进入同一个页面，在图片都加载过的情况下，反复进入退出查看内存状况，如果持续增加，则说明有泄漏。

- 第三方 MLeaksFinder

##内存泄漏的常见场景
###1. CF类型内存
###2. MRC 使用release/autorelease
###3. ARC下的循环引用,block循环引用

#runloop是什么？

> ###RunLoop是一个对象，这个对象在循环中用来处理程序运行过程中出现的各种事件（比如说触摸事件、UI刷新事件、定时器事件、Selector事件），从而保持程序的持续运行；而且在没有事件处理的时候，会进入睡眠模式，从而节省CPU资源，提高程序性能。
> 
- 这种模型通常被称作 Event Loop。 Event Loop 在很多系统和框架里都有实现，比如 Node.js 的事件处理，比如 Windows 程序的消息循环，再比如 OSX/iOS 里的 RunLoop。实现这种模型的关键点在于：如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。
- RunLoop管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。


#专场动画

#响应者链条

在iOS中凡是继承自UIResponder的对象都能够接收并处理事件。
UIResponder一般分为三种事件，触摸事件，加速计事件以及远程控制事件。

#Filie's Owner

#CoreData SQLite比较

#数据库升级怎么做?

#多线程下载一个大文件怎么做? 2GB的文件
- 多线程（线程池）
  - 先完成的线程继续去下载文件
  - 
- 断点续传 Range ,code = 206
- 分割文件(根据线程数分割）
 - 多个小文件
- 设计拼接数据结构
- 数据校验完整性(md5 hash)

#Storyboard还是纯代码布局？各有什么优缺点?
- 多个Storyboard的引用 ，采用 Storyboard Reference
- 

#NSArray如果用Strong修饰有什么注意事项？

#xib自定义View在storyboard中的使用
- 1.拖一个UIView，class设为自定义View的类型
- 2.重写- (void)awakeFromNib
- 3.重写- (void)setFrame:(CGRect)frame (Autolayout)

#TCP、UDP的区别

#HTTP、RPC

#引入第三方框架时选择 Embedded 是什么区别？
- .dylib(.so)和.framework
  动态链接库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。
- .a 和 .framework
  静态链接库：链接时完整地拷贝至可执行文件中，被多次使用就有多份冗余拷贝。
- framework为什么既是静态库又是动态库
  系统的.framework是动态库，我们自己建立的.framework是静态库。
- a与.framework有什么区别

> .a是一个纯二进制文件，.framework中除了有二进制文件之外还有资源文件。
.a文件不能直接使用，至少要有.h文件配合，.framework文件可以直接使用。
.a + .h + sourceFile = .framework。

> ###建议用.framework.

- 为什么要使用静态库

> 方便共享代码，便于合理使用。
> 实现iOS程序的模块化。可以把固定的业务模块化成静态库.
> 和别人分享你的代码库，但不想让别人看到你代码的实现。
> 开发第三方sdk的需要。

 
- 制作静态库时的几点注意：
  - 1 注意理解：无论是.a静态库还.framework静态库，我们需要的都是二进制文件+.h+其它资源文件的形式，不同的是，.a本身就是二进制文件，需要我们自己配上.h和其它文件才能使用，而.framework本身已经包含了.h和其它文件，可以直接使用。

  - 2 图片资源的处理：两种静态库，一般都是把图片文件单独的放在一个.bundle文件中，一般.bundle的名字和.a或.framework的名字相同。.bundle文件很好弄，新建一个文件夹，把它改名为.bundle就可以了，右键，显示包内容可以向其中添加图片资源。

  - 3 category是我们实际开发项目中经常用到的，把category打成静态库是没有问题的，但是在用这个静态库的工程中，调用category中的方法时会有找不到该方法的运行时错误（selector not recognized），解决办法是：在使用静态库的工程中配置other linker flags的值为-ObjC。

  - 4 如果一个静态库很复杂，需要暴露的.h比较多的话，就可以在静态库的内部创建一个.h文件（一般这个.h文件的名字和静态库的名字相同），然后把所有需要暴露出来的.h文件都集中放在这个.h文件中，而那些原本需要暴露的.h都不需要再暴露了，只需要把.h暴露出来就可以了。




#如何定制自己的Framework


#Objective-C异常处理机制

> Objective-C中处理异常是依赖于NSException实现的，它是异常处理的基类，它是一个实体类，而并非一个抽象类，所以你可以直接使用它或者继承它扩展使用

`@try/@catch(NSException *e)/@finally`
> Objective-C是C语言的扩充，它的异常处理机制是通过C标准库提供两个特殊的函数setjmp()和longjmp()函数实现的。如果对C的异常处理机制和setjmp、longjmp函数不了解的，建议先阅读：C语言异常处理机制。

```
objc_exception_try_enter、objc_exception_extract、objc_exception_throw、objc_exception_try_exit
```

```
             * _setjmp是C的函数，用于保存当前程序现场。
             * _setjmp需要传入一个jmp_buf参数，保存当前需要用到的寄存器的值。
             * _setjmp()它能返回两次，第一次是初始化时，返回0，第二次遇到_longjmp()函数调用会返回，返回值由_longjmp的第二个参数决定。
             * 如果对_setjmp()和_longjmp()概念不太了解的，请参考C语言的异常处理机制。
             *
             * 下面_setjmp()初始化返回0，然后执行if{}中也就是@try{}中的代码。
             
```
>
>

#iOS JSBridge的原理

#JSPatch热更新原理

JSPatch 能做到通过 JS 调用和改写 OC 方法最根本的原因是 Objective-C 是动态语言，OC 上所有方法的调用/类的生成都通过 Objective-C Runtime 在运行时进行，我们可以通过类名/方法名反射得到相应的类和方法：

Class class = NSClassFromString("UIViewController");
id viewController = [[class alloc] init];
SEL selector = NSSelectorFromString("viewDidLoad");
[viewController performSelector:selector];
也可以替换某个类的方法为新的实现：

static void newViewDidLoad(id slf, SEL sel) {}
class_replaceMethod(class, selector, newViewDidLoad, @"");
还可以新注册一个类，为类添加方法：

Class cls = objc_allocateClassPair(superCls, "JPObject", 0);
objc_registerClassPair(cls);
class_addMethod(cls, selector, implement, typedesc);
对于 Objective-C 对象模型和动态消息发送的原理已有很多文章阐述得很详细，这里就不详细阐述了。理论上你可以在运行时通过类名/方法名调用到任何 OC 方法，替换任何类的实现以及新增任意类。所以 JSPatch 的基本原理就是：JS 传递字符串给 OC，OC 通过 Runtime 接口调用和替换 OC 方法。

#Core Data与SQLite比较


|SQLite|Core Data|
|:-:|:-:|
|轻量级,占用内存和磁盘空间小|占用内存和磁盘空间大|
||读取速度比SQLite块|
|跨平台|iOS独有|
|学习周期短,有SQL基础即可|学习周期长，要熟悉一套API|
|需要重新封装对象|支持对象操作|
|自定义升级过程|升级方便|
|/|支持iCloud|
|/|支持Undo,Redo|
|第三方库:FMDB|第三方库:MagicalRecord|
||面向对象编程|
||KVC\KVO的支持|


- CoreData的优点

对象图管理
惰性加载的支持（faulting and uniquing）
面向对象的编程，直观易用
良好的多线程支持(注意不是线程安全的)
支持redo和undo
NSFetchedResultController使得与tableview结合编程变得很容易
KVC与KVO的支持
ICould的支持
Apple 推荐的存储方式，API在不断升级

- CoreData的缺点

 - 学习成本高，要很久才能得心应手
 - 对象Schema改变后，数据迁移比较棘手（当然也支持）
 - 对于一次大量更新删除等操作效率较低（因为每次都要先取到内存里）
 - 对主键的支持要自己去维护（CoreData 通过objectID来唯一确定对象）
 - 占用内存会高一些（为了维护ManagedContext，为了跟踪对象变化）

- 使用SQLite的优点

 - 学习成本相对较低
 - 直接使用SQLite引擎，对一次大批量数据的操作性能较好（只需要一个SQL语句即可）。
 - 占用内存较少
 - 更加轻量
 - Android和WP也支持

- 使用SQLite的缺点
 
 - 复杂的对象关系要自己维护
 - 没有对象变化跟踪
 - 并不是真正的面向对象编程

#Weex 跨平台方案

![](https://upload-images.jianshu.io/upload_images/201701-51b1eec568b4aecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#自定义Framework

#组件化设计 

- Protocol注册方案
- URL 注册方案
- Target-Action runtime调用方案+ Category