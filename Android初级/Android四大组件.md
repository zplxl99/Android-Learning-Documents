---
title: Android四大组件
tags: Android,Activity
grammar_cjkRuby: true
---

## Android四大组件之Activity

 1. Activity初识
 
 　　官方的定义：Activity 是一个应用组件，用户可与其提供的屏幕进行交互，以执行拨打电话、拍摄照片、发送电子邮件或查看地图等操作。通俗的来说：这就是一个机器与人交互的媒介。
	 
　　 一个应用是有若干个Activity组成的。首次启动应用时出现的那个Activity称为主Activity（在AndroidManiftest.xml中会记为：`<action android:name="android.intent.action.MAIN" />`）。Activity的存储结构为：栈（Activity Stack）
 
 
 2. Activity的生命周期
 
     先上一张最经典的图：
	 
	![enter description here](https://developer.android.google.cn/images/activity_lifecycle.png)
	
	2.1.各个方法职责：
	　onCreate()：必须的方法， 是应用启动的最开始，在Activity生命周期中有且仅会触发一次。
	
	　onResume()：每次启动Activity所需要初始化的地方。此时Acvitiy处于可见状态。
	
	　onPause()：系统将发出信号，指出 Activity 将暂时暂停，且用户可能将焦点返回到您的 Activity，或者应用在多窗口模式下运行。此时Activity处于部分可见状态。
	
	　onStop()：释放几乎所有用户不使用时不需要的资源。此时Activity处于不可见的状态。
	
	　onDestory()：销毁当前的这个Activity。
	
	2.2.很明显能看到这个流程图有四条线：
	　A.onCreate()--> onStart()-->onResume()-->onPause()-->onStop()-->onDestory()    ----------这是一个Activity从启动到结束的正常完整流程
	　B.onCreate()--> onStart()-->onResume()-->onPause()-->onResume()    ----------多用于界面往返
	　C.onCreate()--> onStart()-->onResume()-->onPause()-->onStop()-->onRestart()-->onStart()-->onResume()   ----------多用从后台调起界面
   　 D.onCreate()--> onStart()-->onResume()-->onPause()-->onStop()-->onCreate()   ----------多用App进程被kill之后重新进入Activity？这里还需要check
	
	2.3.常见的流程如下：

　　a.应用启动之后进入到主界面：
		　　  onCreate() --> onStart() --> onResume()
		  
　　b.点击back键返回：
	     　　 onPause() --> onStop() --> onDestory()
　　　再次进入应用：
　　　　　onCreate() --> onStart() --> onResume()
 
 　　c.点击Home键退出：
　　　　　onPause() --> onStop()
　　　再次进入应用：
　　　　　onRestart() --> onStart() --> onResume()
		 
　　d.应用内界面的挑战（设Activity A跳转到Activity B）：
　　Activity A： onCreate() --> onStart() --> onResume() --> onPause()                                        --> onStop() 
　　Activity B：                                                                           onCreate() --> onStart() --> onResume()
	    

　　2.４.Activity的状态保存：

　　android给了两个函数来实现：一个是onSaveInstanceState() ,用来save Activity state；另一个是onRestoreInstanceState()，用来Restore Activity state。
	   
　　A.onSaveInstaceState()
　　　调用时机：在活动**可能被杀死之前**调用此方法，以便在将来某个时间返回时它可以恢复其状态。

　　**注意：在Android P及以后，这个方法都是在onStop()方法之后调用，而在Android P之前，这个方法肯定在onStop()方法之前，但是和onPause()的前后时序无法保证。**

　　B.onRestoreInstanceState()
　　　调用时机：当从以前保存的状态重新初始化的时段，在onStart()方法之后调用。默认实现执行先前被onSaveInstanceState()冻结的任何视图状态的恢复。这个方法是介于onStart()方法和onPostCreate()方法之间

　　2.５.Activity的其他状态：
	 还有几个不常用的状态方法：onPostCreate()和onPostResume()

 3. Activity启动方式
　　从Activity的生命周期的流程上可以看到，Activity的存储结构是为堆栈（先进后出）。 有两种方式来控制行为：一种是使用manifest.xml；另一种是使用Intent flag。
	 
　　设有Activity A、Activity B、Activity C，我们打印出Activity的任务栈（adb shell dumpsys activity）：
	 
　　　A.默认为Standard，就是每次调起Activity即往stack中压栈。如下：
	 
![这是从A跳转到B，再从B跳转到C的正常的入栈，可以看到先进的在栈底，后进的在栈顶。](./images/QQ截图20181206201216.png)
	 
	 
　　　B.singleTop，栈顶唯一，即若这个Activity实例在栈顶，则不会再继续创建新的实例，而是直接调用。
	 对应的是FLAG_ACTIVITY_SINGLE_TOP，堆栈如下：
	 
 ![这是从A跳转到B，再从B跳转到B的栈内情况，B中的launch mode为singleTop。可以看到实例B有且仅有一个](./images/QQ截图20181206221549.png)


![若未设置，则多次创建实例B，堆栈内情况](./images/QQ截图20181206221847.png)
	
　　　C.singleTask，每创建一个实例，就会将其放入到一个新的堆栈中。对应FLAG_ACTIVITY_NEW_TASK
	
　　　　注意：这里光是去设置launch mode之后并没有效果，堆栈如下：
	
![B的启动方式设置为singleTask](./images/QQ截图20181206235016.png)
	
　　　　这里需要定义taskAffinity，然后可以看到B和C在一个task里，而A在另一个task里，堆栈如下：
	
![定义B的android:taskAffinity=“rose.android.com.newtask”](./images/QQ截图20181206235723.png)
	
　　　D.singleInstance，和singleTask类似，只不过一个堆栈中有且仅有一个Actvity实例：
	
![](./images/QQ截图20181207010530.png)
	 

 4. Activity启动流程(留待后续)
　　　![](./images/QQ截图20181222184445.png) 
 



## Android四大组件之Service

 1. Service的定义：
　　Service 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。
 
　　A.服务不是一个单独的进程，服务对象并不运行在自身的进程中；
　　B.服务也不是一个线程；
	 
 　　因此服务本身实际上非常简单，提供两个主要特征：
　　　a.告知消停有需要在后台处理的信息，对应于Context.startService()的 调用，这会告知系统去安排运行服务，直至服务或者其他停止。
　　　b.将某些功能暴露给其他应用程序，对应于Context.bindService()的调用，它允许对服务进行长期连接以便与服务进行交互。

 2. Service的生命周期：
 
    系统运行服务的方式有两种：
	
　　A. Context.startService()：
	 
　　当应用组件（如 Activity）通过调用 startService() 启动服务时，服务即处于“启动”状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。 已启动的服务通常是执行单一操作，而且不会将结果返回给调用方。例如，它可能通过网络下载或上传文件。 操作完成后，服务会自行停止运行。
	
　　　　　onCreate() --> onStartCommand(Intent,int,int) --> Context.stopService/stopSelf()
	   
　　　对于已经启动的服务，有三种决定他们运行方式的操作模式：
	   
　　　a.START_STICKY：用于根据需要显示启动和停止的服务；如果系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但不会重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。这适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。

　　　b.START_NOT_STICKY：用于在处理发送给它们的任何命令时应该只保持运行的服务。如果系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

　　　c.START_REDELIVER_INTENT：如果系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。任何挂起 Intent 均依次传递。这适用于主动执行应该立即恢复的作业（例如下载文件）的服务。
	
　　B.Context.bindService()：

　　当应用组件通过调用 bindService() 绑定到服务时，服务即处于“绑定”状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。 仅当与另一个应用组件绑定时，绑定服务才会运行。 多个组件可以同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。
　　　只要建立连接，服务将保持运行（无论客户端是否保留对服务的Ibinder的引用）
　　　　　onCreate() --> onBind()
	
 　　 附上google官方的服务生命周期图：
 　　 ![](./images/service_lifecycle.png)
  
 3. Service的扩展-IntentService：
　　这是 Service 的子类，它使用工作线程逐一处理所有启动请求。如果您不要求服务同时处理多个请求，这是最好的选择。 您只需实现 onHandleIntent() 方法即可，该方法会接收每个启动请求的 Intent，使您能够执行后台工作。

IntentService 执行以下操作：

　　A.创建默认的工作线程，用于在应用的主线程外执行传递给 onStartCommand() 的所有 Intent。
　　B.创建工作队列，用于将 Intent 逐一传递给 onHandleIntent() 实现，这样您就永远不必担心多线程问题。
　　C.在处理完所有启动请求后停止服务，因此您永远不必调用 stopSelf()。
　　D.提供 onBind() 的默认实现（返回 null）。
　　E.提供 onStartCommand() 的默认实现，可将 Intent 依次发送到工作队列和 onHandleIntent() 实现。

　　只需要一个构造函数和一个 onHandleIntent() 实现即可
　　![](./images/QQ截图20181222183204_1.png)
 

## Android四大组件之ContentProvider

 1. ContentProvider的定义：
 
　　内容提供者是应用程序的主要构建快之一，为应用程序提供内容，封装数据并通过单个ContentProvider接口将其提供给应用程序。
　　仅在多个应用应用程序之间共享数据时，才需要内容提供者。
　　当通过ContentResolver发出请求时，系统检查给定的URI的权限，并将请求传递给权限注册的内容提供者。内容提供者可以解释它降妖的URI的其余部分。
 
 2. 常用接口：

　　onCreate()：来初始化提供者;
　　query（Uri，String []，Bundle，CancellationSignal）：它将数据返回给调用者插入（Uri，ContentValues），将新数据插入内容提供程序

　　update（Uri，ContentValues，String，String []）：用于更新内容提供程序中的现有数据
　　delete（Uri，String，String []）：从内容提供程序中删除数据
　　getType（Uri）：它返回内容提供程序中的MIME类型的数据
  
  

## Android四大组件之Broadcast Receiver

1. Broadcast ：
1.1.Broadcast定义：
　　顾名思义，广播是指Android应用程序从Android系统或其他应用程序发送或接收某些消息。当有感兴趣的事件发生时，就会发送广播。
　　例如：系统启动时或者设备开始充电时；也可以是应用程序发送自定义广播，来通知其他应用程序他们可能感兴趣的内容。
　　一般而言，广播可以用作跨应用程序和普通用户流程之外的消息传递系统，但不建议滥用机会响应广播并在后台运行可能导致系统性能降低。
1.2.Broadcast的分类：
　　有两种：一是自定义广播；一是系统广播。系统广播可参看Android SDK中BROADCAST_ACTIONGS.TXT文件。每个广播动作都有一个与之相关的常量字段。系统广播的优化：
 　　A. Android 7.0 (API =24)及更高版本
 　　 　　不发送以下广播：ACTION_NEW_PICTURE、ACTION_NEW_VIDEO。此外针对API大于等于24的应用必须使用registerReceiver(BroadcastReceiver,intentFilter)注册CONNECTIVITY_ACTION广播。在manifest中声明接收器不起作用。
 　　B. Android 8.0 (API =26)开始
 　　 　　系统对Manifest中声明的接收器施加了额外限制，无法为大多数隐式广播声明接收方。
 　　C. Android 9.0 (API =28)开始
 　　 　　NETWORK_STATE_CHANGED_ACTION广播不会接收有关用户位置或个人身份识别数据的信息。此外，来自wifi的系统广播不包含SSID、BSSID、连接信息或扫描结果，要获取这些信息需要调用getConnectionInfo()。
1.3.具有权限的Broadcast：
　　可以指定Broadcast的权限参数。只有那些已经在其Manifest中请求带有标签许可的接收者才可以接收广播。
　　为此首先，app需要带有对应的权限：
 
2. BroadcastReceiver：
　　应用程序可以通过两种方式来接收广播：一种是静态注册（通过mainfest声明的接收器）；另一种是动态注册（通过context注册的接收器）。
1.1.静态注册：
　　A.在AndroidManifest.xml中指定<receiver>元素。其中intent-filter指定接收者订阅的广播动作。
　　 ![](./images/QQ截图20181222180311.png)
　　B.定义子类BroadcastReceiver并实现onReceive(Context,Intent)方法。
　　 ![](./images/QQ截图20181222180438.png)
1.2.动态注册：
　　A.创建一个BroadcastReceiver实例；
　　B.创建一个IntentFilter，并通过调用registerReceiver(BroadcastReceiver,IntentFilter)来注册接收器。注意：本地广播的注册方式为：LocalBroadcastManager.registerReceiver(BroadcastReceiver,IntentFilter);
　　![](./images/QQ截图20181222180700.png)
　　C.要停止广播，则调用unregisterReceiver。当不再需要接收器或者上下文不再有效时，务必取消注册的接收器。注册/取消注册接收器的位置：onCreate()<-->onDestory() ，onResume() <--> onPause()；

3. send BroadcastReceiver：
　　Android提供三种方式来发送广播：
3.1.sendOrderedBroadcast(Intent,String)：
　　一次向一个接收器发送广播。当每个接收器一次执行时，才可将结果传递给下一个接收器或者也可以终止广播。
　　需要匹配相同的intent-filter，并通过android:priority属性控制优先。具有相同的priority的接收器将以任意顺序进行。
3.2.sendBroadcast(Intent)：
　　以未定义的顺序向所有接收器发送广播。更有效，但也意味着接收器无法从其他接收器读取结果或中止广播。
3.3.LocalBroadcastManager.sendBroadcast：
　　将广播发送到与发送方位于同一个应用程序的接收方。即为本地广播。以未定义的顺序向所有接收器发送广播。更有效，但也意味着接收器无法从其他接收器读取结果或中止广播。
  
4. 安全考虑因素：
4.1.若不需要向应用程序外部的组件发送广播，则使用支持库中提供的LocalBroadcastManager发送和接收本地广播；
4.2.若许多应用已注册在其manifest文件中接收相同的广播，则可能导致系统启动大量应用，从而影响设置性能和用户体验。为避免如此，优先使用动态转注册；
4.3.不要使用隐式意图广播敏感信息。任何注册接收广播的应用都可以读取该信息。有三种方式可控制谁可以接收广播：A.在发送广播的时候指定权限；B.在发送广播的时候指定包含setPackage(String)的包；C.使用LocalBroadcastManager发送本地广播；
4.4.当注册接收器时，任何应用都可以向应用接收器发送潜在的恶意广播，有三种方式可以限制应用收到的广播：A.在注册广播接收器时指定权限；B.对于静态注册的接收器，可以在manifest.xml中将android:exported属性设置为“false”，则接收方不接收来自应用程序之外来源的广播；C.使用LocalBroadcastManager发送本地广播；
4.5.广播的Action命名是全局的，因此需要保证广播名称的唯一性；
4.6.接收者的onReceive(Context,Intent)方法运行在主线程上，所以该方法要快速执行并返回。因此建议：一是在接收器的onReceive()方法中调用goAsync()并将BroadcastReceiver.PendingResult传递给后台线程。使得onReceive()返回后广播保持活动的状态。即使这样也需要保证在10s以内完成广播，否则会出现ANR；另一个是使用JobSchedule；