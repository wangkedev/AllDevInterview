# 那些著名或非著名的iOS面试题（下）

### 1. Runtime

Objective-C 是面相运行时的语言（runtime oriented language），就是说它会尽可能的把编译和链接时要执行的逻辑延迟到运行时。这就给了你很大的灵活性，你可以按需要把消息重定向给合适的对象，你甚 至可以交换方法的实现，等等。

RunTime简称运行时。就是系统在运行的时候的一些机制，其中最主要的是消息机制。OC的函数调用成为消息发送。属于动态调用过程。在编译的时候并不能决定真正调用哪个函数（事实证明，在编 译阶段，OC可以调用任何函数，即使这个函数并未实现，只要申明过就不会报错。而C语言在编译阶段就会报错）。只有在真正运行的时候才会根据函数的名称找 到对应的函数来调用。

以下面的代码为例：

[obj makeText];

其中obj是一个对象，makeText是一个函数名称。对于这样一个简单的调用。在编译时RunTime会将上述代码转化成

objc_msgSend(obj,@selector(makeText));

首先，编译器将代码[obj makeText];转化为objc_msgSend(obj, @selector (makeText));，在objc_msgSend函数中。首先通过obj的isa指针找到obj对应的class。在Class中先去cache中 通过SEL查找对应函数method（猜测cache中method列表是以SEL为key通过hash表来存储的，这样能提高函数查找速度），若 cache中未找到。再去methodList中查找，若methodlist中未找到，则取superClass中查找。若能找到，则将method加 入到cache中，以方便下次查找，并通过method中的函数指针跳转到对应的函数中去执行。

Objective-C Runtime 是什么？

Objective-C 的 Runtime 是一个运行时库（Runtime Library），它是一个主要使用 C 和汇编写的库，为 C 添加了面相对象的能力并创造了 Objective-C。这就是说它在类信息（Class information） 中被加载，完成所有的方法分发，方法转发，等等。Objective-C runtime 创建了所有需要的结构体，让 Objective-C 的面相对象编程变为可能。

Method Swizzling 原理

在Objective-C中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。利用Objective-C的动态特性，可以实现在运行时偷换selector对应的方法实现，达到给方法挂钩的目的。每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现。

我们可以利用 method_exchangeImplementations 来交换2个方法中的IMP，

我们可以利用 class_replaceMethod 来修改类，

我们可以利用 method_setImplementation 来直接设置某个方法的IMP，……

归根结底，都是偷换了selector的IMP。

### 2. GCD实现1，2并行和3串行和45串行，4，5是并行。即3依赖1，2的执行，45依赖3的执行。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fvrm7h3r1hj30fd03hglm.jpg)

​											关系

队列组的方式

```objective-c
- (void) methodone{
dispatch_group_t group = dispatch_group_create();
 
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"%d",1);
});
 
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"%d",2);
});
 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"3");
 
    dispatch_group_t group1 = dispatch_group_create();
 
    dispatch_group_async(group1, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"%d",4);
    });
 
    dispatch_group_async(group1, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"%d",5);
    });
 
});
 
}
```



串行队列：队列中的任务只会顺序执行

dispatch_queue_t q = dispatch_queue_create(“....”, dispatch_queue_serial);

并行队列： 队列中的任务通常会并发执行。 　　　　　　　

dispatch_queue_t q = dispatch_queue_create("......", dispatch_queue_concurrent);

全局队列：是系统开发的，直接拿过来用就可以；与并行队列类似，但调试时，无法确认操作所在队列　。

dispatch_queue_t q = dispatch_get_global_queue(dispatch_queue_priority_default, 0);

主队列：每一个应用开发程序对应唯一一个主队列，直接get即可；在多线程开发中，使用主队列更新UI。

dispatch_queue_t q = dispatch_get_main_queue();

主队列是GCD自带的串行队列，会在主线程中执行。异步全局并发队列 开启新线程，并发执行。

并行队列里开启同步任务是有执行顺序的，只有异步才没有顺序。

串行队列开启异步任务，是有顺序的。

串行队列开启异步任务后嵌套同步任务造成死锁。

### 3. 深浅复制和属性为copy，strong值的变化问题

浅复制：只复制指向对象的指针，而不复制引用对象本身。对于浅复制来说，A和A_copy指向的是同一个内存资源，复制的只不个是一个指针，对象本身资源还是只有一份，那如果我们对A_copy执行了修改操作，那么发现A引用的对象同样被修改了。深复制就好理解了，内存中存在了两份独立对象本身。

在Objective-C中并不是所有的对象都支持Copy，MutableCopy，遵守NSCopying协议的类才可以发送Copy消息，遵守NSMutableCopying协议的类才可以发送MutableCopy消息。

[immutableObject copy] // 浅拷贝

[immutableObject mutableCopy] //深拷贝

[mutableObject copy] //深拷贝

[mutableObject mutableCopy] //深拷贝

属性设为copy,指定此属性的值不可更改，防止可变字符串更改自身的值的时候不会影响到对象属性（如NSString,NSArray,NSDictionary）的值。strong此属性的指会随着变化而变化。copy是内容拷贝，strong是指针拷贝。

### 4.NSTimer创建后，会在哪个线程运行。

用scheduledTimerWithTimeInterval创建的，在哪个线程创建就会被加入哪个线程的RunLoop中就运行在哪个线程。

自己创建的Timer，加入到哪个线程的RunLoop中就运行在哪个线程。

### 5. KVO，NSNotification，delegate及block区别

KVO就是cocoa框架实现的观察者模式，一般同KVC搭配使用，通过KVO可以监测一个值的变化，比如View的高度变化。是一对多的关系，一个值的变化会通知所有的观察者。

NSNotification是通知，也是一对多的使用场景。在某些情况下，KVO和NSNotification是一样的，都是状态变化之后告知对方。NSNotification的特点，就是需要被观察者先主动发出通知，然后观察者注册监听后再来进行响应，比KVO多了发送通知的一步，但是其优点是监听不局限于属性的变化，还可以对多种多样的状态变化进行监听，监听范围广，使用也更灵活。

delegate 是代理，就是我不想做的事情交给别人做。比如狗需要吃饭，就通过delegate通知主人，主人就会给他做饭、盛饭、倒水，这些操作，这些狗都不需要关心，只需要调用delegate（代理人）就可以了，由其他类完成所需要的操作。所以delegate是一对一关系。

block是delegate的另一种形式，是函数式编程的一种形式。使用场景跟delegate一样，相比delegate更灵活，而且代理的实现更直观。

KVO一般的使用场景是数据，需求是数据变化，比如股票价格变化，我们一般使用KVO（观察者模式）。delegate一般的使用场景是行为，需求是需要别人帮我做一件事情，比如买卖股票，我们一般使用delegate。Notification一般是进行全局通知，比如利好消息一出，通知大家去买入。delegate是强关联，就是委托和代理双方互相知道，你委托别人买股票你就需要知道经纪人，经纪人也不要知道自己的顾客。Notification是弱关联，利好消息发出，你不需要知道是谁发的也可以做出相应的反应，同理发消息的人也不需要知道接收的人也可以正常发出消息。

### 6. 如何让计时器调用一个类方法

计时器只能调用实例方法，但是可以在这个实例方法里面调用静态方法。

使用计时器需要注意，计时器一定要加入RunLoop中，并且选好model才能运行。scheduledTimerWithTimeInterval方法创建一个计时器并加入到RunLoop中所以可以直接使用。

如果计时器的repeats选择YES说明这个计时器会重复执行，一定要在合适的时机调用计时器的invalid。不能在dealloc中调用，因为一旦设置为repeats 为yes，计时器会强持有self，导致dealloc永远不会被调用，这个类就永远无法被释放。比如可以在viewDidDisappear中调用，这样当类需要被回收的时候就可以正常进入dealloc中了。

### 7. 调用一个类的静态方法需不需要release？

静态方法，就是类方法，不需要，类方法对象放在autorelease中

###8. static作用？

（1）函数体内 static 变量的作用范围为该函数体，不同于 auto 变量，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值；

（2）在模块内的 static 全局变量可以被模块内所用函数访问，但不能被模块外其它函数访问；

（3）在模块内的 static 函数只可被这一模块内的其它函数调用，这个函数的使用范围被限制在声明

它的模块内；

（4）在类中的 static 成员变量属于整个类所拥有，对类的所有对象只有一份拷贝；

（5）在类中的 static 成员函数属于整个类所拥有，这个函数不接收 this 指针，因而只能访问类的static 成员变量。

### 9. NSObject的load和initialize方法

##### load和initialize的共同特点

在不考虑开发者主动使用的情况下，系统最多会调用一次

如果父类和子类都被调用，父类的调用一定在子类之前

都是为了应用运行提前创建合适的运行环境

在使用时都不要过重地依赖于这两个方法，除非真正必要

##### load和initialize的区别

load方法

调用时机比较早，运行环境有不确定因素。具体说来，在iOS上通常就是App启动时进行加载，但当load调用的时候，并不能保证所有类都加载完成且可用，必要时还要自己负责做auto release处理。对于有依赖关系的两个库中，被依赖的类的load会优先调用。但在一个库之内，调用顺序是不确定的。

对于一个类而言，没有load方法实现就不会调用，不会考虑对NSObject的继承。

一个类的load方法不用写明[super load]，父类就会收到调用，并且在子类之前。

Category的load也会收到调用，但顺序上在主类的load调用之后。

不会直接触发initialize的调用。

##### initialize方法相关要点

initialize的自然调用是在第一次主动使用当前类的时候。

在initialize方法收到调用时，运行环境基本健全。

initialize的运行过程中是能保证线程安全的。

和load不同，即使子类不实现initialize方法，会把父类的实现继承过来调用一遍。注意的是在此之前，父类的方法已经被执行过一次了，同样不需要super调用。

由于initialize的这些特点，使得其应用比load要略微广泛一些。可用来做一些初始化工作，或者单例模式的一种实现方案。

### 10. 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

不能向编译后得到的类中增加实例变量；

能向运行时创建的类中添加实例变量；

因为编译后的类已经注册在 runtime 中，类结构体中的 objc_ivar_list 实例变量的链表 和 instance_size 实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；

运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。