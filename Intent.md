# Intent 学习笔记

1. Intent包括4大部分， ComponentName, Action, Category, Data

2. ComponentName不空的Intent称为显式Intent( Explicit Intent )， 因为这类Intent直接指出了接收Intent的类，ComponentName类包括两个部分，Package Name和Class Name, Intent有一个构造函数需要传入一个Context和一个Class类，其中Context是用来获取PackageName的，也就是说构建Intent的时候不一定需要Activity

3. 如果没有定义ComponentName，这类Intent就变成了隐式Intent( Implicit Intent )，由于没有明确指明Intent的接收者，因此需要系统去寻找和该Intent匹配的类来处理，这种情况下就需要结合IntentFilter食用了。通过同时在Intent和IntentFilter中定义Action，Category和Data，来完成他们之间的匹配。

4. 如果Intent是隐式Intent而且该Intent用在startActivity中，**一定要在调用startActivity()之前调用该intent的resolveActivity()方法来看一下有没有Activity能处理这个Intent，如果没有匹配到Activity的话，App会崩溃的啊！**，如果该隐式Intent是用在startService中的话，well，**永远不要用隐式Intent启动Service！！！**，Intent接收方用户不可见，是非常不安全的一种方式，而且新的Android版本也不会支持这样的启动方式

4. 只有同时满足Action，Category和Data的匹配规则，该Intent才会被交给注册了IntentFilter的类进行处理。

5. **Action过滤规则**：IntentFilter中可以定义0个或多个接收的Action，只要IntentFilter中所包含的Action是IntentFilter中定义Action的子集就算匹配，如果IntentFilter中定义0个Action，则没有Intent可以与之匹配

6. **Category过滤规则**：与Action规则非常类似，Intent中包含的Category也必须是IntentFilter中定义Category的子集，区别是当IntentFilter和Intent都没有定义Category( 定义0个Category )的时候，Action会匹配失败，但是Category会匹配成功，即没有定义Category的Intent，任何IntentFilter都可以对它进行处理。**有坑高能预警！startActivity()和startActivityForResult()方法会对传入的Intent进行处理，为Intent添加CATEGORY_DEFAULT**，所以如果自定义的Category为空的Intent结果没有匹配到预期的IntentFilter，就可以大胆猜一下是不是掉这个坑里了。

7. **Data过滤规则**：Data主要包括两个部分，Uri和MimeType，匹配规则很简单，Uri和MimeType完全匹配(content和file的Schema除外)，即：如果Intent没有定义Uri和MimeType，则同样没有定义Uri和MimeType的IntentFilter会匹配成功；如果Intent只定义了Uri没定义MimeType，同样定义方式的IntentFilter且Uri匹配的会匹配成功；其他的也类似。**需要注意：当Intent定义Uri的Schema为content或者file且IntentFilter并没有定义Uri的时候，Uri部分会被认为匹配成功**
	
	- Uri匹配规则：按照Schema -> Authority -> Path 的顺序进行匹配，IntentFilter与Intent完全匹配或者IntentFilter为空则认为匹配成功( IntentFilter的Schema如果为空的话，参考上面 )
	- Uri定义规则：IntentFilter在定义Uri部分的过滤规则时，需要遵循以下要求
		- 如果没定义Schema，Host定义也没用
		- 如果没定义Host， port定义了也没用
		- 如果Schema和Host都没定义，那么path定义了也没用
		
8. Intent其他玩法：在startActivity的时候，可以通过Intent的静态方法createChoose，把原本的Intent包装一层，从而强制弹出应用选择框

9. PendingIntent：这个类是Intent类的一个包装类，专门用来让其他应用使用其中包装的Intent或者定时使用该Intent的( 典型应用：NotificationManager， App Widget，还有AlarmManager等 )

> 由于Intent设计之初就是为了让例如Activity，Service或者BroadcastReceiver之类的App Component来接收的，Intent类可以直接通过startActivity()，startService()，或者sendBroadcast()之类的方法来指定接受者，PendingIntent作为包装类，没有这类方法，所以在使用PendingIntent的时候，需要通过getActivity()，getService()或者getBroadcast()方法来制定所包装的Intent期待的接收者