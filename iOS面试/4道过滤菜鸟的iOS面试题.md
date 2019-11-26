#  4道过滤菜鸟的iOS面试题



原文链接：http://www.jianshu.com/p/c687110e552c

### 1. struct和class的区别

swift中，class是引用类型，struct是值类型。值类型在传递和赋值时将进行复制，而引用类型则只会使用引用对象的一个"指向"。所以他们两者之间的区别就是两个类型的区别。

class有这几个功能struct没有的：

- class可以继承，这样子类可以使用父类的特性和方法
- 类型转换可以在runtime的时候检查和解释一个实例的类型
- 可以用deinit来释放资源
- 一个类可以被多次引用

struct也有这样几个优势：

- 结构较小，适用于复制操作，相比于一个class的实例被多次引用更加安全
- 无须担心内存memory leak或者多线程冲突问题

顺便提一下，array在swift中是用struct实现的。Apple重写过一次array，然后复制就是深度拷贝了。猜测复制是类似参照那样，通过栈上指向堆上位置的指针来实现的。而对于它的复制操作，也是在相对空间较为宽裕的堆上来完成的，所以性能上还是不错的。

下面引用猫神OneV的博客：

var arr = [0,0,0]

var newArr = arr

arr[0] = 1

//Check arr and newArr

arr //[1, 0, 0]

newArr // before beta3:[1, 0, 0], after beta3:[0, 0, 0]

所以可以猜测其实在背后 Array和 Dictionary的行为并不是像其他 struct 那样简单的在栈上分配，而是类似参照那样，通过栈上指向堆上位置的指针来实现的。而对于它的复制操作，也是在相对空间较为宽裕的堆上来完成的。当然，现在还无法（或者说很难）拿到最后的汇编码，所以这只是一个猜测而已。

补充：C语言中，struct与的class的区别：struct只是作为一种复杂数据类型定义，不能用于面向对象编程。

C++中，struct和class的区别：对于成员访问权限以及继承方式，class中默认的是private的，而struct中则是public的。class还可以用于表示模板类型，struct则不行。

### 2. 介绍一下观察者模式

观察者模式(Observer Pattern)：定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。在IOS中典型的推模型实现方式为NSNotificationCenter和KVO。

1. 观察者Observer，通过NSNotificationCenter的addObserver:selector:name:object接口来注册对某一类型通知感兴趣。在注册时候一定要注意，NSNotificationCenter不会对观察者进行引用计数+1的操作，我们在程序中释放观察者的时候，一定要去报从center中将其注销了。

1. 通知中心NSNotificationCenter，通知的枢纽。

1. 被观察的对象，通过postNotificationName:object:userInfo:发送某一类型通知，广播改变。

1. 通知对象NSNotification，当有通知来的时候，Center会调用观察者注册的接口来广播通知，同时传递存储着更改内容的NSNotification对象。

#### KVO

KVO的全称是Key-Value Observer，即键值观察。是一种没有中心枢纽的观察者模式的实现方式。一个主题对象管理所有依赖于它的观察者对象，并且在自身状态发生改变的时候主动通知观察者对象。

1. 注册观察者[object addObserver:self forKeyPath:property options:NSKeyValueObservingOptionNew context:]。
2. 更改主题对象属性的值，即触发发送更改的通知。
3. 在制定的回调函数中，处理收到的更改通知。
4. 注销观察者 [object removeObserver:self forKeyPath:property]。

### 3. 在一个HTTPS连接的网站里，输入账号密码点击登录后，到服务器返回这个请求前，中间经历了什么

这个非常得深非常得广，我来大概说一下。

HTTPS加密流程

![](https://ws2.sinaimg.cn/large/006tNc79ly1fvrj86248dj30i00fkdgj.jpg)

1. 客户端会打包一个请求，包括url，端口啊，你的账号密码等等。账号密码登陆应该用的是Post方式，所以相关的用户信息会被加载到body里面。这个请求应该包含三个方面：网络地址，协议，资源路径。注意，这里是HTTPS，就是HTTP + SSL / TLS，在HTTP上又加了一层处理加密信息的模块（相当于是个锁）。

2一般会先请求DNS服务器。DNS服务器负责将你的网络地址解析成IP地址，这个IP地址对应网上一台机器。这其中可能发生Hosts Hijack和ISP failure的问题。

3.协议是获取资源的方式HTTP，FTP，UDP，不同协议有不同的格式，有些是process-to-process的，有些是host-to-host的。

4.客户端会和服务器的端口之间建立一个socket连接，socket一般都是以file descriptor的方式解析请求。

5.服务器端接收到请求。服务器端会有一套数字证书（相当于是个钥匙），这个证书会先返回给客户端。客户端会解析证书，相当于用钥匙（证书）把锁（内容）锁上（生成私匙），接着再传送加密信息。

6.服务器端接收到加密信息（私匙）之后，会进行解密，并把要返回的数据进行对称加密返回到客户端。假如路径不对，会出现404的错误。

7.一般访问服务器之前可能会访问一下proxy。这玩意是个代理，有时候当防火墙用，有时候当cache使。如果后台是reverse-proxy结构，那么实际上有多个web服务器藏在proxy之后按需处理请求，而你访问的永远是proxy，这样可以解决过载问题。

8.有时候访问完web服务器后还要访问一下file服务器，主要是请求数据库里的一些信息。

9.服务器将相应打包，直接或通过proxy（大多数时候）返回给客户端。客户端会用刚刚生成的私匙进行解密，将内容显示在浏览器上。

10.HTTPS加密过程详解请去https原理：[证书传递、验证和数据加密、解密过程解析](http://blog.csdn.net/clh604/article/details/22179907)

### 4. 在一个app中间有一个button，在你手触摸屏幕点击后，到这个button收到点击事件，中间发生了什么

响应链大概有以下几个步骤

1. 设备将touch到的UITouch和UIEvent对象打包, 放到当前活动的Application的事件队列中

1. 单例的UIApplication会从事件队列中取出触摸事件并传递给单例UIWindow

1. UIWindow使用hitTest:withEvent:方法查找touch操作的所在的视图view

RunLoop这边我大概讲一下

1. 主线程的RunLoop被唤醒
2. 通知Observer，处理Timer和Source 0
3. Springboard接受touch event之后转给App进程
4. RunLoop处理Source 1，Source1 就会触发回调，并调用_UIApplicationHandleEventQueue() 进行应用内部的分发。
5. RunLoop处理完毕进入睡眠，此前会释放旧的autorelease pool并新建一个autorelease pool

深挖请去[深入理解RunLoop](http://blog.ibireme.com/2015/05/18/runloop/)

UIResponder是UIView的父类，UIView是UIControl的父类。

声明一下，第3题依然有很大缺陷，不过因为深挖的地方太多，本文不可能完全兼顾，只能抛砖引玉。另外文章的目的是以面试题为引进行学习，所以写得有点多，可能与面试技巧和时间有冲突。

