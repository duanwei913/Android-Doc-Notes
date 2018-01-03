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