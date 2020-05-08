# iOS-
此贴是给自己用来记录知识回顾的blog。

基础篇
1.kvo的具体实现流程？访问成员变量（类似self->age）会触发KVO吗？kvc会触发kvo吗？kvo的两个核心调用方式是？
kvo 是oc一种观察者模式的又一实现，使用isa混写（isa-swizzling）来实现kvo。
当注册一个观察一个属性的时候，系统会为我们在运行时创建一个NSKVONotifying_class，将A的isa指针指向NSKVONotifying_A,NSKVONotifying_A重写Setter方法来达到监听值的变化。
kvo会触发，核心调用willchangeValueForKey 和 didChangeValueForKey

2.Category和 extension分别的使用场合和特点是什么？

Category 运行时决议
为系统类添加一些方法以及属性
公开一些系统私有方法
分解臃肿的类

extension 编译时决议
添加私有属性
添加私有方法
添加私有成员变量

3.iOS响应链和事件分发。
手指点击屏幕后，applicat进行事件传递到当前window，通过hittest：withevent 返回最终响应的视图，在hitTest：withEvent内部是根据pointInside：withEvent来找到响应的UIWindow，然后根据subviews通过倒叙遍历 也是通过hitTest：WithEvent 以及pointInSide：withEvent 返回最终响应的视图

Runtime篇
1.isa指针是什么？里面有哪些特殊的位数？什么是TaggedPointer的优化？
isa分为isa指针和非isa指针，isa指针的值是class地址，

2.class的底层结构是什么样的？
class 结构有 superclass， cache_t ,class_data_bits_t（？？？）
class_data_bits_t 是对class_rw_t的封装
class_rw_t 包含 class_ro_t，properties，methods，protocols
class_ro_t 包含 name，ivars，properties，methods，protocols

3.method_t里包含了什么？
method_t 包含了 SEL name， const char *types（参数返回值合集） ， imp

4.super的本质是什么？
super的本质是objc_msgSendSuper

5.oc的消息机制有几步？
一共有三步，
1.resolveInstanceMethod：
2.forwardingTargetForSelector：
3.methodSignatureForSelector: ->forwardInvcation


6.如何防止类似 unrecongnized selector的错误？ _objc_msgForward能干什么？
交换原有的methodSignatureForSelector：和 forwardInvcation方法，在methodSignagtureForSelector方法中监听方法有没有实现，没有实现将方法动态添加到实现的方法中，
_objc_msgForward是消息转发，指在当前类如果没有找到方法实现就会尝试做消息转发。

7.runtime有哪些应用？方法替换（method - Swizzling）有什么缺点？如何安全的进行方法替换？
动态添加方法，交换方法等。
Method swizzling is not atomic
Method swizzling不是原子性操作。如果在+load方法里面写，是没有问题的，但是如果写在+initialize方法中就会出现一些奇怪的问题。
Changes behavior of un-owned code
如果你在一个类中重写一个方法，并且不调用super方法，你可能会导致一些问题出现。在大多数情况下，super方法是期望被调用的（除非有特殊说明）。如果你是用同样的思想来进行Swizzling，可能就会引起很多问题。如果你不调用原始的方法实现，那么你Swizzling改变的越多就越不安全。
Possible naming conflicts
命名冲突是程序开发中经常遇到的一个问题。我们经常在类别中的前缀类名称和方法名称。不幸的是，命名冲突是在我们程序中的像一种瘟疫。一般我们用方案一的方式来写
安全替换
方案一：通常用在对_cmd没有严格要求，或者不用_cmd来做事情的场合；通常用方法名前加前缀的方式来避免命名冲突；考虑当前类未实现要被swizzle的方法的应用场景。
方案二：要对某私有类来swizzling方法时，常采用；通常用方法名前加前缀的方式来避免命名冲突；考虑当前类未实现要被swizzle的方法的应用场景。
方案三：标准的swizzling方案，通常用在对_cmd有严格要求，或者用_cmd来做事情的场合；不需要考虑命名冲突；要用static修饰的变量来保存相关IMP，考虑函数和变量的命名冲突。

RunLoop 篇

1.Runloop的本质是什么？
Runloop是通过内部维护的事件循环来对事件/消息进行管理的一个对象。
没有消息需要处理时，休眠以避免资源占用
有消息需要处理时，立刻被唤醒
没有消息处理时是将用户态切换到内核态

main函数为什么保持不退出？
是因为调用一个UIApplicationMain，函数内部会启动主线程的Runloop。

2.Runloop和线程是什么关系？
Runloop和线程是一一对应关系，根据Runloop结构可以得出
3.Runloop的底层数据结构是什么样的？有几种 运行模式（mode）？每个运行模式下面的CFRunloopMode 是哪些？他们分别是什么？
NSRunloop 和 CFRunloop，NSRunloop是对CFRunloop的封装。
CFRunloop包含
pthread，
currentMode，
modes，
commonModes，
cmomonModeItems
4.Runloop的监听状态有哪几种？

5.Runloop的工作流程大概是什么样的？

6.Runloop有哪些应用？

7.多线程，异步执行（async）一个performSelector会执行吗？如果加上afterDelay呢？
不一定执行，performSelector 需要在runloop下才能执行，要看当前线程是否有runloop。

锁篇
1.你知道iOS有哪些锁？性能分别怎么样？

2.自旋锁和互斥锁怎么选择？


内存管理篇
1.引用计数器怎么实现的？weak怎么实现的？sideTable的底层结构是怎么样的？weak指针做了什么操作？

2.AutoReleasePool（自动释放池）的底层实现是什么？他怎么实现及时释放的？子线程的释放时机是怎样的？

3.对象的release是怎么处理的？

网络篇 
1.http中的get和post方式有什么区别？

2.Https连接建立流程是怎样的？

3.TCP和UDP的区别

4.简述TCP慢开始的过程

5.客户端怎样避免DNS劫持
