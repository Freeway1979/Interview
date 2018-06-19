Android面试复习

#启动Activity的几种方法有什么区别？
- `standard`：标准模式，这是系统默认的默认，也就是说你不设置Activity的launchMode时，默认的就是standard。在这种模式下，`每次启动一个Activity都会重新创建一个新的实例`，不管这个实例是否存在。在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈，但是这是有条件的，前提是启动它的Activity不能是singleInstance，因为singleInstance只能独立于一个任务栈中，不能有其他的Activity实例。比如非singleInstance模式的Activity A启动了标准模式的Activity B，那么Activity B就会进入到Activity A所在的任务栈。
- `singleTop`：栈顶复用模式。在这种模式下，如果Activity已经在任务栈的栈顶了，当再次启动同一个Activity的时候，这个Activity不会被重新创建，而且它的`onNewIntent()`方法会被调用，但是它的onCreate()、onStart()方法不会被调用。此模式下的Activity也会进入启动它的非singleInstance模式的Activity所在的任务栈中。如果不是栈顶，跟standard一样。
- `singleTask`：栈内复用模式。在这种模式下，只要Activity存在栈内，那么多次启动这个Activity都不会重新创建实例，系统会调用它的`onNewIntent()`方法。此外有个需要注意的地方：singleTask有clear top的效果，也就是说会将其以上的Activity全部出栈。
- `singleInstance`：这是singleTask的一种加强模式，除了singleTask所有特性以外，具有此模式的Activity只能单独位于一个任务栈中。


#讲解你的项目架构


#常用的设计模式
- Singleton
  
  > 工具类
  
  > 使用enum构建线程安全的单例
- Builder 
  > 构建复杂的对象
- Proxy
 
 > 缩略图下载
- Adapter

 > ListView Adapter

- FactoryMethod
- Observer(Notification/Broadcast)
- Mediator(仲裁者模式)
-  

#正向代理、反向代理
- 正向代理

  > 正向代理 是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。
  > ###`服务端不知道客户端是谁`。
  > 正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率。
  > 正向代理允许客户端通过它访问任意网站并且`隐藏客户端自身`，因此你必须采取安全措施以确保仅为经过授权的客户端提供服务。
  
- 反向代理
  
  > 对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理 的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容 原本就是它自己的一样。
  
 > ###`客户端不知道谁为他服务`。
 > 反向代理的典型用途是将 防火墙后面的服务器提供给Internet用户访问。反向代理还可以为后端的多台服务器提供`负载平衡`，或为后端较慢的服务器提供`缓冲服务`。
 > 反向代理还可以启用高级URL策略和管理技术，从而使处于不同web服务器系统的web页面同时存在于同一个URL空间下。
 > 反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。
  
  
#代理和动态代理
###1.静态代理
- 共同接口
- 真实对象
- 代理对象

-------
-  优点：扩展原功能，不侵入原代码。
-  缺点：面对多个功能类似的接口时，就得设计多个代理，导致代理的膨胀。

###2.动态代理
  
  通过使用动态代理，我们可以通过在运行时，动态生成一个持有RealObject、并实现代理接口的Proxy，同时注入我们相同的扩展逻辑。哪怕你要代理的RealObject是不同的对象，甚至代理不同的方法，都可以动过动态代理，来扩展功能。

> ###利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

- proxy的创建都是自动的并且是在运行期生成的。

- 使用动态代理，需要将要扩展的功能写在一个`InvocationHandler` 实现类里： 

```
public class DynamicProxyHandler implements InvocationHandler {
    private Object realObject;

    public DynamicProxyHandler(Object realObject) {
        this.realObject = realObject;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //代理扩展逻辑
        System.out.println("proxy do");

        return method.invoke(realObject, args);
    }
}

public static void main(String[] args) {
        RealObject realObject = new RealObject();
        Action proxy = (Action) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{Action.class}, new DynamicProxyHandler(realObject));
        proxy.doSomething();
}
``` 
实现 **InvocationHandler**
  
  **bind/invoke**
  
- AOP编程: Spring MVC,Structs的拦截器
 
- cglib动态代理
> 利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
  
  
  

 > ####JDK代理是不需要以来第三方的库，只要要JDK环境就可以进行代理，它有几个要求
* 实现InvocationHandler 
* 使用Proxy.newProxyInstance产生代理对象
* 被代理的对象必须要实现接口

> ###CGLib 必须依赖于CGLib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承但是针对接口编程的环境下推荐使用JDK的代理
在Hibernate中的拦截器其实现考虑到不需要其他接口的条件Hibernate中的相关代理采用的是CGLib来执行。

  
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
  
#Service的两种启动方式

- startService

 - 1.定义一个类继承Service
 - 2.在Manifest.xml文件中配置该Service
 - 3.使用Context的startService(Intent)方法启动该Service
 - 4.不再使用时，调用stopService(Intent)方法停止该服务

> 使用这种start方式启动的Service的生命周期如下：
onCreate()--->onStartCommand()（onStart()方法已过时） ---> onDestory()

> 说明：如果服务已经开启，不会重复的执行onCreate()， 而是会调用onStart()和onStartCommand()。
服务停止的时候调用 onDestory()。服务只会被停止一次。

> 特点：一旦服务开启跟调用者(开启者)就没有任何关系了。
开启者退出了，开启者挂了，服务还在后台长期的运行。
开启者不能调用服务里面的方法。

- bindService

#Service和IntentService的区别

  - 1.定义一个类继承Service
  - 2.在Manifest.xml文件中配置该Service
  - 3.使用Context的bindService(Intent, ServiceConnection, int)方法启动该Service
  - 4.不再使用时，调用unbindService(ServiceConnection)方法停止该服务
 
 > 使用这种start方式启动的Service的生命周期如下：
onCreate() --->onBind()--->onunbind()--->onDestory()

 > 注意：绑定服务不会调用onstart()或者onstartcommand()方法

 > 特点：bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。
绑定者可以调用服务里面的方法。

> 绑定本地服务调用方法的步骤：

- 在服务的内部创建一个内部类 提供一个方法，可以间接调用服务的方法
- 实现服务的onbind方法，返回的就是这个内部类
- 在activity 绑定服务。bindService();
- 在服务成功绑定的回调方法onServiceConnected， 会传递过来一个 IBinder对象
- 强制类型转化为自定义的接口类型，调用接口里面的方法。

#[广播](https://www.jianshu.com/p/ea5e233d9f43)

```
public class MyBroadCastReceiver extends BroadcastReceiver   
{  
   @Override  
   public void onReceive(Context context, Intent intent)   
   {   
       //在这里可以写相应的逻辑来实现一些功能
       //可以从Intent中获取数据、还可以调用BroadcastReceiver的getResultData()获取数据
   }   
}

```

- 静态注册广播

  > 常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。
 
- 动态注册广播
 
 > 不是常驻型广播，也就是说广播跟随程序的生命周期。
 
- 无序广播：所有的接收者都会接收事件，不可以被拦截，不可以被修改。

> mContext.sendBroadcast(Intent)或mContext.sendBroadcast(Intent, String)发送的是无序广播(后者加了权限)；

- 有序广播：按照优先级，一级一级的向下传递，接收者可以修改广播数据，也可以终止广播事件。

> 通过mContext.sendOrderedBroadcast(Intent, String, BroadCastReceiver, Handler, int, String, Bundle)发送的是有序广播。



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

#ABTest

AB测试是为Web或App界面或流程制作两个（A/B）或多个（A/B/n）版本，在同一时间维度，分别让组成成分相同（相似）的访客群组随机的访问这些版本，收集各群组的用户体验数据和业务数据，最后分析评估出最好版本正式采用。

