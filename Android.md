Android面试复习

#启动Activity的几种方法有什么区别？
- standard
  创建新对象
- singleInstance
  只有一个实例
- singleTask
  新建一个栈
- singleTop
  如果在顶端就不创建直接用，否则创建。

#讲解你的项目架构

#常用的设计模式

#代理和动态代理
###1.静态代理
###2.动态代理
  
  实现 **InvocationHandler**
  
  **bind/invoke**
  
  AOP编程: Spring MVC,Structs的拦截器
  
#观察者模式

#Adapter模式

#仲裁模式

#单例（线程安全）
- 线程安全单例
  - 1.饥饿式
  - 2.懒汉式
  - 3.静态成员类
  - 4.**枚举** 
- 内存泄漏问题
  单例传入Context会引起内存泄漏，无法释放，改成传入ApplicationContext
  或者参数不要持有，仅仅在单例的方法中参入，而不是构造函数中传入。

#简单工厂有什么特点？



#Android多设备UI适配
- dp/sp
- 百分比

#Android热更新原理

#Android内存泄漏

### [内存泄漏](https://www.jianshu.com/p/ab4a7e353076)
> 如果一个不需要再使用的对象（强引用)仍然被其他对象持有引用，造成该对象无法被系统回收。

### 1.什么情况下会造成内存泄漏?
- 单例导致内存泄漏
- 静态变量导致内存泄漏
- 非静态内部类导致内存泄漏	
  + Handler/Thread/AsyncTask 
  
  > 静态内部类+弱引用
  mHandler.removeCallbacksAndMessages(null);
- 缓存没有释放不再引用的对象
- 未取消注册或回调导致内存泄露
- Timer和TimerTask导致内存泄露
  > cancel timer
- 集合中的对象未清理造成内存泄露
- 资源未关闭或释放导致内存泄露
- 属性动画造成内存泄露
- WebView造成内存泄露

### 2.如何调试分析？

- WeTest 腾讯
- in
- Android MAT
- 
- LeakCanary Square
- 

#Android Lint
- 静态代码检查
- 静态代码检查框架
- Lint的优点

1. 功能强大，Lint支持Java源文件、class文件、资源文件、Gradle等文件的检查。
2. 扩展性强，支持开发自定义Lint规则。
3. 配套工具完善，Android Studio、Android Gradle插件原生支持Lint工具。
4. Lint专为Android设计，原生提供了几百个实用的Android相关检查规则。
5. 有Google官方的支持，会和Android开发工具一起升级完善。

- 参考 [美图技术文章](https://tech.meituan.com/waimai-android-lint.html)

#Activity传递大数据怎么办？
- Bundle最大约1M 
  TransactionTooLargeException
  The Binder transaction failed because it was too large
- 传递大数据（比如bitmap）尽量避免采用bundle传递
  - 写文件或数据库
  - 写全局缓存（需要和调用者在一个进程）
  - 单例（需要和调用者在一个进程)

#Service和IntentService的区别

###1.Service

> **Service 是长期运行在后台的应用程序组件。**

> Service 不是一个单独的进程，它和应用程序在同一个进程中，Service 也不是一个线程,它和线程没有任何关系，所以它不能直接处理耗时操作。如果直接把耗时操作放在 Service 的 onStartCommand() 中，很容易引起 ANR .如果有耗时操作就必须开启一个单独的线程来处理。

###2.IntentService

> IntentService 是继承于 Service 并处理异步请求的一个类，在 IntentService 内有一个工作线程来处理耗时操作，启动 IntentService 的方式和启动传统 Service 一样，同时，当任务执行完后，IntentService 会自动停止，而不需要我们去手动控制。另外，可以启动 IntentService 多次，而每一个耗时操作会以工作队列的方式在IntentService 的 **onHandleIntent** 回调方法中执行，并且，每次只会执行一个工作线程，执行完第一个再执行第二个，以此类推。

> 而且，**所有请求都在一个单线程中**，不会阻塞应用程序的主线程（UI Thread），同一时间只处理一个请求。
> 
> IntentService是一个通过Context.startService(Intent)启动可以处理异步请求的Service,使用时你只需要继承IntentService和重写其中的onHandleIntent(Intent)方法接收一个Intent对象,在适当的时候会停止自己(一般在工作完成的时候). 所有的请求的处理都在一个工作线程中完成,它们会交替执行(但不会阻塞主线程的执行),一次只能执行一个请求。

> 这是一个基于消息的服务,每次启动该服务并不是马上处理你的工作,而是首先会创建对应的Looper,Handler并且在MessageQueue中添加的附带客户Intent的Message对象,当Looper发现有Message的时候接着得到Intent对象通过在onHandleIntent((Intent)msg.obj)中调用你的处理程序.处理完后即会停止自己的服务.意思是Intent的生命周期跟你的处理的任务是一致的.所以这个类用下载任务中非常好,下载任务结束后服务自身就会结束退出. 
> 

#应用内广播

`LocalBroadcastManager`
避免广播敏感信息，被其他恶意的receiver接收到.

#Handler一定要在主线程实例化吗?new Handler()和new Handler(Looper.getMainLooper())的区别

|场景|           当前在主线程            |当前在工作线程|
|:-:|:-:|:-:|
|需要刷新UI|Handler handler = new Handler();|Handler handler = new Handler(Looper.getMainLooper());|
|不用刷新ui,只是处理消息|Handler handler = new Handler();|Looper.prepare(); Handler handler = new Handler();Looper.loop();或Handler handler = new Handler(Looper.getMainLooper());|

### 1. `Handler handler = new Handler();`默认使用当前线程的looper
### 2. 如果需要刷新UI，那么1的代码需要在主线程创建。如果不是在主线程可以通过传入参数指定主线程的looper。
`Handler handler = new Handler(Looper.getMainLooper());`

### 3.不用刷新ui,只是处理消息。 当前线程如果是主线程的话，`Handler handler = new Handler();`不是主线程的话，`Looper.prepare(); Handler handler = new Handler();Looper.loop();`或者`Handler handler = new Handler(Looper.getMainLooper());`

#HandlerThread的特点

- HandlerThread将loop转到子线程中处理，说白了就是将分担MainLooper的工作量，降低了主线程的压力，使主界面更流畅。

- 开启一个线程起到多个线程的作用。处理任务是串行执行，按消息发送顺序进行处理。HandlerThread本质是一个线程，在线程内部，代码是串行处理的。

- 但是由于每一个任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么就会导致后续的任务都会被延迟处理。

- HandlerThread拥有自己的消息队列，它不会干扰或阻塞UI线程。

- 对于网络IO操作，HandlerThread并不适合，因为它只有一个线程，还得排队一个一个等着。

#Android架构设计

- [Android Architecture Components](https://blog.csdn.net/hubinqiang/article/details/73012336?utm_source=tuicool&utm_medium=referral)
- 反编译其他APP来学习
- 根据APP业务特点来确定架构
- 分层设计
1. UI (Activity/Fragment/Views



#UI相关
##1.不能使用 ScrollView 包裹 ListView/GridView/ExpandableListVIew

> 因为这 样会把 ListView 的所有 Item 都加载到内存中，要消耗巨大的内存和 cpu 去绘制图面。
> 
> 推荐使用 **NestedScrollView**。

#Android更新UI的几种方法

###1.Handler+View.invalidate+Thread+Runnable

###2.View.postInvalidate+Thread+Runnable

> postInvalidate就是把message(MSG_INVALIDATE,view)丢给主线程mHandler,mHandler收到后view.invalidate

###3.Handler+Worker Thread

> 自定义主线程Handler，从Worker Thread发送消息 

###4.Worker Thread 中runOnUiThread

```
public final void runOnUiThread(Runnable action) {  
       if (Thread.currentThread() != mUiThread) {  
           mHandler.post(action);//将Runnable Post到消息队列，由内部的mHandler来处理，实际上也是Handler的处理方式  
       } else {  
           action.run();//已经在UI线程，直接运行。  
       }  
   }  
```

###5.AysncTask更新UI


#插件化
