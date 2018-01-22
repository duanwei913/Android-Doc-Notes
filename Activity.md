# Activity学习笔记

**1. 什么是Activity**

Activity是Android应用中最基本的元素，直观来讲，可以与App使用过程中某一个瞬间屏幕上的内容对应（i.e. 一屏内容就是一个Activity， **只是直观讲，这种说法不严谨，比如当有弹窗的时候屏幕上可能出现多于一个Activity**）

**2. 配置Activity**

需要理解，Android系统中的每个App都运行在自己单独的沙盒中以保证安全，而一个App (Android Application) 中所包含的一些核心元素 (比如Activity) 必须向系统报备，以实现上层程序逻辑和底层的硬件等系统资源的一些交互 (个人理解的原因)。 为了实现该报备的目的，每个App都必须有一个AndroidManifest.xml文件，文件中以<application\>为根元素的块包含App内部组件的全部信息

所有定义的Activity必须都在这里进行声明 (报备), 并定义一些与该Activity有关的属性或配置 (比如该Activity的屏幕方向，Activity的名称等等)，可以在<Activity\>标签内部再嵌套其他标签

**3. Activity生命周期理解**

以ActivityA启动ActivityB，之后ActivityB调用finish()方法为例
整个流程为

**ActivityA:**onPause() -> **ActivityB:**onCreate() -> **ActivityB:**onStart() -> **ActivityB:**onResume() -> **ActivityA:**onStop() ( ActivityA启动ActivityB完成，此时ActivityB在前台并和用户交互，ActivityA 不在屏幕显示)

**ActivityB:**onPause() -> **ActivityA:**onRestart() ->
**ActivityA:**onStart() -> **ActivityA:**onResume() -> **ActivityB:**onStop() -> **ActivityB:**onDestroy() (此时ActivityA在前台并和用户进行交互，ActivityB被完全销毁)

由此可以看出，在Activity切换的过程中，在要离开的Activity的onPause()方法返回之后，新创建的Activity要一直执行到onResume()方法，之后才会继续执行要离开的Activity的后续回调，所以，onPause()方法是个非常重要的方法，别在里面搞耗时的状态存储操作，否则会严重阻塞UI，这和不要在onResume里做初始化操作一样。

**4. 对于onSaveInstanceState()和onRestoreInstanceState()的理解**

> 有些时候，由于一些用户行为( 例如在App运行过程中旋转屏幕 ), 或者是一些系统行为( 例如系统内存不足 )导致不在前台的Activity被暂时回收，为了在用户浏览回该Activity的时候，当时的现场可以被正确恢复，系统提供了这样一个回调，在onSaveInstanceState()中保存一些在App生命周期内该Activity所需要的场景数据

onSaveInstanceState()的默认实现为根据id保存当前的View Hierarchy ( **Tips:给每个View还有ViewGroup分配一个id看起来是个好习惯** )，该方法默认会在onStop()之前调用，但是其和onPause()的调用关系具有随机性

onSaveInstanceState()会把这些数据保存在一个Bundle中，后面在这个Activity被重建的时候，这个Bundle会被同时传递给onCreate()和onRestoreInstanceState(), 和onSaveInstanceState()相对的，onRestoreInstanceState()会在onStart()之后调用

**5. Task，Activity，Application，Process**

官方文档对于Task的解释是：Task是为了完成一项任务而创建的一组Activity，这些Acitivity以栈的形式进行组合，也就是所谓的返回栈(Back Stack)。为了区别不同的Task，每个Task都应该有一个id，但由于Task只是一个概念(没办法直接new一个Task出来)，所以Task的id必须由它的内容来指定(**换句话说：不存在空Task，每个Task至少会包含一个Activity，而Activity作为Application基本组件之一，也就是说，每个Task至少包含一个Application**)

Application通常指的就是我们常说的App，它是一个可安装的apk文件，通常在桌面包含一个启动图标。我们所说的Activity，Service，以及BroadCast Receiver和Content Provider等都是Application的基本组件，Application就是由这些组件构成的一个概念。默认情况下，整个Application都会运行在一个Android Process，同时也是一个Standard Linux Process里，但是这并不绝对，可以在AndroidManifest中为每个组件单独定义process，也就是说，**Application和Process没有一一对应关系**

> 综上，个人的理解是，Task，Application和Process之间没有相互包含关系，一个Task可能包含多个Application，一个Application里也可能包含多个Task，一个Application可以运行在多个Process里

> 除了Process是实际对应Linux Process之外，Application和Task都是一个抽象概念，他们代表了看问题的不同角度。Task可以类比为一个用户场景(Use Case)，比如拍照之后微信分享这个场景，可能会涉及到微信和相机两个Application；而一些相对独立的用户场景，比如在某个Activity中加载网页，虽然该Activity仍然属于我们的Application，但是在用户场景上，我们把它放到浏览器的Task内可能会更加合理

* TaskAffinity

	由于Task的id必须由其内容来定义，所以在AndroidManifest中，Application和Activity都提供了一个taskAffinity参数，用来指定他们所属的Task，如果Activity未指定，默认使用Application的，如果Application也没制定，默认会使用Package name来作为Application所属的Task id(**这样也保证了在默认的情况下，所有的Activity都属于一个Task**)


* launchMode

	在每个Activity都属于一个Task的情形下，就可以讨论每个Activity该如何被启动了。每个Activity的启动过程包含两方，from 和 to，即被启动的Activity和请求启动其他Activity的来源Activity，Android系统针对这两方对launchMode进行了定义，需要注意，如果AndroidManifest和Intent的设置冲突的话(**冲突的粒度可以很细,未冲突的部分，系统会将AndroidManifest和Intent中定义的行为进行组合**)，以Intent中的设置为准
	
	* AndroidManifest：launchMode Attribute
		* **standard**：在From Activity所属Task创建新实例
		* **singleTop**：同standard，当该Task的Back Stack栈顶已经是To Activity了，该Activity的onNewIntent被调用，不再创建新实例
		* **singleTask**：首先在系统中寻找和To Activity的Task Affinity相同的Task，如果没找到，创建新Task并创建To Activity新实例；如果找到了，在该Activity的Back Stack中寻找To Activity实例，如果没找到，创建新实例并入栈，如果找到了，将该实例之上的所有Activity全都出栈，并调用该实例onNewIntent
		* **singleInstance**：基本同singleTask，但是当To Activity再创建其他Activity的时候，不管新的Activity是什么launchMode，都会在一个新的Task内创建，也就是说，定义为singleInstance的Activity，其所在的Task永远只能包含它自己
		
	* Intent：setFlags (AndroidManifest：launchMode=standard)
		* **FLAG_ACTIVITY_NEW_TASK**：该flag请求系统在一个新Task中启动To Activity，系统在检测到该flag的时候，会首先搜索To Activity定义的taskAffinity，如果该值和From Activity的taskAffinity相同的话，该flag就会直接失效，系统继续用To Activity在AndroidManifest中注册的启动方式进行启动，如果From和To Activity的taskAffinity不相同，那么会查找是否存在目标Task，如果不存在，创建新Task并实例化To Activity，如果存在的话，如果该Task里面已经包含了To Activity的实例，那么系统只把该Task提到前台；如果不存在的话，还是要实例化To Activity并入栈(**注意：这里只针对在AndroidManifest中定义launchMode为Standard的情形，如果在AndroidManifest中有定义其他launchMode，所有和该flag不冲突的行为仍然会执行，比如，如果定义为singleTask，那么启动To Activity的时候会自动加上clearTop效果**)
		
		* **FLAG_ACTIVITY_SINGLE_TOP**：和AndroidManifest中额singleTop行为完全一致，但是当AndroidManifest中定义为singleTask且taskAffinity不同时，会执行singleTask的逻辑(**请再仔细体会细粒度的冲突的含义**，冲突针对的是不同的启动方式背后的每一条行为，不是启动方式本身)
		
		* **FLAG_ACTIVITY_CLEAR_TOP**：该方法通常是FLAG_ACTIVITY_NEW_TASK的一个补充，当该Activity实例存在于目标Task的时候，将该Activity之上的所有Activity全部出栈
		
		* 另外，还有MULTIPLE_TASK， FORWARD_RESULT之类的flag，后续参考Intent文档进行了解(**最好养成习惯，当具体业务场景中出现启动Component相关的需求并且不好解决的时候(比如一个完整的登录流程涉及三个Activity，如何startActivityForResult)，记得查询一下Intent中的相关flag，很多坑已经被填了，不要重复造轮子**)
		
* AllowTaskReparenting

	Task对于Activity和Application的包含关系，可以用Parent-Child的模式来进行描述。每个Activity，除非设置launchMode为singleTask，否则当该Activity的taskAffinity和其对应的Application的taskAffinity不一致时，如果设置allowTaskReparenting，其就会处于一种"不稳定"状态。具体表现为，当这类Activity的实例存在时，如果该Activity的目标Task从另外的Application被启动的时候，该Activity就会自动开启"找妈"模式，被从原来的Task转移到目标Task

* Application生命周期

由于Application是有一组Component构成的集合，所以当所有的Component都被杀死之后，Application也跟着被杀死，否则该Application会一直存活。由于不同的Component可以跑在不同的Process上，所以只有当所有Component所在的Process都被杀死之后，Application才会被杀死。**在有些情况下，比如低内存，系统会对当前所有的Process依照重要性进行排序，杀死低优先级Process，保证高优先级Process**

> Process优先级排序：
> 
> 	- 前台进程(Foreground Process): Component正在执行, 如Activity正在执行onResume, Service在执行回调, BroadCast Receiver在执行onReceive之类的
>
> 	- 可见进程(Visible Process): Component仍然能被用户感知,比如Activity执行onPause, Service以startForeground的方式启动,或者Service正在被系统调用进行某些用户可感知行为, 比如换壁纸
>
>	- 服务进程(Service Process): Service以startService的方式启动之后，其所在进程就是服务进程
>
>	- 缓存进程(Cached Process): 用户不可感知，系统随时可能杀死该进程，比如Activity在调用了onStop之后


**6. 数据传递**

在Activity间跳转的时候，可以在Intent中用putExtra的方式塞一些数据进去，除了通用基本类型之外，可以用Bundle和Parcel来封装一些基本数据类型和复杂数据结构，但是需要注意坑：

	- putExtra的时候，KEY的管理最好用一个专门的Constants类来进行，否则代码维护成本可能会很高
	
	- 虽然通过实现Parcelable接口理论上可以传递任何数据，但是一定要小心数据量过大的问题，官方推荐数据量在50KB以下，否则在7.0及以上的系统会受到TransactionTooLargeException，在7.0以下会传递数据失败
	
	- 如果是跨进程传数据，不要传自定义Parcel对象，根本无法保证数据一致性的
