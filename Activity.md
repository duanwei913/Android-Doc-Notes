# Activity学习笔记

**1. 什么是Activity**

Activity是Android应用中最基本的元素，直观来讲，可以与App使用过程中某一个瞬间屏幕上的内容对应（i.e. 一屏内容就是一个Activity， **只是直观讲，这种说法不严谨，比如当有弹窗的时候屏幕上可能出现多于一个Activity**）

**2. 配置Activity**

需要理解，Android系统中的每个App都运行在自己单独的沙盒中以保证安全，而一个App (Android Application) 中所包含的一些核心元素 (比如Activity) 必须向系统报备，以实现上层程序逻辑和底层的硬件等系统资源的一些交互 (个人理解的原因)。 为了实现该报备的目的，每个App都必须有一个AndroidManifest.xml文件，文件中以<application\>为根元素的块包含App内部组件的全部信息

所有定义的Activity必须都在这里进行声明 (报备), 并定义一些与该Activity有关的属性或配置 (比如该Activity的屏幕方向，Activity的名称等等)，可以在<Activity\>标签内部再嵌套其他标签