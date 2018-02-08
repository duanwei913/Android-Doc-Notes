# Fragment 笔记

**1. Fragment设计逻辑**

Activity在安卓中一般对应着一个完整的页面，但是当App需要适配多个不同尺寸的屏幕的时候，对于一个页面的定义可能并不相同。比如经典的列表-详情页模式，在小屏手机上会对应着两个Activity，但是在很大屏幕的手机或者pad上，可能会将这两个Activity整合成一个页面( 否则会出现大面积的空白或者不一致的UI )，用传统的Activity的方式就不得不为不同尺寸的设备编写两套完全不可复用的代码。

为了解决这个问题，Android针对页面内的元素进行了二次划分，将功能相对完善的一个模块作为一个片段进行封装，从而可以实现页面内元素更加灵活的可复用的组合方式，也就是Fragment
