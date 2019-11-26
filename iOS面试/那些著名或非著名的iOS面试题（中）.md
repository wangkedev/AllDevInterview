## 那些著名或非著名的iOS面试题（中）

###  1. 反转二叉树，不用递归

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
```

递归方式：

```javascript
public class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        root.left = invertTree(root.left);
        root.right = invertTree(root.right);
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;
        return root;
    }
}
```

OC 实现：

```objective-c
/** 
 * 翻转二叉树（又叫：二叉树的镜像） 
 *
 * @param rootNode 根节点
 *
 * @return 翻转后的树根节点（其实就是原二叉树的根节点） 
 */
 + (BinaryTreeNode *)invertBinaryTree:(BinaryTreeNode *)rootNode {
    if (!rootNode) {  return nil; } 
    if (!rootNode.leftNode && !rootNode.rightNode) {  return rootNode; } 
    [self invertBinaryTree:rootNode.leftNode];
    [self invertBinaryTree:rootNode.rightNode]; 
    BinaryTreeNode *tempNode = rootNode.leftNode; 
    rootNode.leftNode = rootNode.rightNode;
    rootNode.rightNode = tempNode; 
    return rootNode;
  }
```

非递归方式：

```objective-c
+ (BinaryTreeNode *)invertBinaryTree:(BinaryTreeNode *)rootNode {
    if (!rootNode) {  return nil; }
    if (!rootNode.leftNode && !rootNode.rightNode) {  return rootNode; }
    NSMutableArray *queueArray = [NSMutableArray array]; //数组当成队列
    [queueArray addObject:rootNode]; //压入根节点
    while (queueArray.count > 0) {
        BinaryTreeNode *node = [queueArray firstObject];
        [queueArray removeObjectAtIndex:0]; //弹出最前面的节点，仿照队列先进先出原则
        BinaryTreeNode *pLeft = node.leftNode;
        node.leftNode = node.rightNode;
        node.rightNode = pLeft;
        if (node.leftNode) {
            [queueArray addObject:node.leftNode];
        }
        if (node.rightNode) {
            [queueArray addObject:node.rightNode];
        }
    }
    return rootNode;
}
```

### 2.写一个单例模式

```objective-c
+ (AccountManager *)sharedManager
{
    static AccountManager *sharedAccountManagerInstance = nil;
    static dispatch_once_t predicate;
    dispatch_once(&predicate, ^{
            sharedAccountManagerInstance = [[self alloc] init]; 
    });
    return sharedAccountManagerInstance;
}
```

3.iOS应用生命周期

**应用程序的状态：**

- Not running未运行：程序没启动。
- Inactive未激活：程序在前台运行，不过没有接收到事件。在没有事件处理情况下程序通常停留在这个状态。
- Active激活：程序在前台运行而且接收到了事件。这也是前台的一个正常的模式。
- Backgroud后台：程序在后台而且能执行代码，大多数程序进入这个状态后会在在这个状态上停留一会。时间到之后会进入挂起状态(Suspended)。有的程序经过特殊的请求后可以长期处于Backgroud状态。
- Suspended挂起：程序在后台不能执行代码。系统会自动把程序变成这个状态而且不会发出通知。当挂起时，程序还是停留在内存中的，当系统内存低时，系统就把挂起的程序清除掉，为前台程序提供更多的内存。

iOS的入口在main.m文件：

```objective-c
int main(int argc, char *argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

main函数的两个参数，iOS中没有用到，包括这两个参数是为了与标准ANSI C保持一致。 UIApplicationMain函数，前两个和main函数一样，重点是后两个。

后两个参数分别表示程序的主要类(principal class)和代理类(delegate class)。如果主要类(principal class)为nil，将从Info.plist中获取，如果Info.plist中不存在对应的key，则默认为UIApplication；如果代理类(delegate class)将在新建工程时创建。

根据UIApplicationMain函数，程序将进入AppDelegate.m，这个文件是xcode新建工程时自动生成的。下面看一下AppDelegate.m文件，这个关乎着应用程序的生命周期。

1、application didFinishLaunchingWithOptions：当应用程序启动时执行，应用程序启动入口，只在应用程序启动时执行一次。若用户直接启动，lauchOptions内无数据,若通过其他方式启动应用，lauchOptions包含对应方式的内容。

2、applicationWillResignActive：在应用程序将要由活动状态切换到非活动状态时候，要执行的委托调用，如 按下 home 按钮，返回主屏幕，或全屏之间切换应用程序等。

3、applicationDidEnterBackground：在应用程序已进入后台程序时，要执行的委托调用。

4、applicationWillEnterForeground：在应用程序将要进入前台时(被激活)，要执行的委托调用，刚好与applicationWillResignActive 方法相对应。

5、applicationDidBecomeActive：在应用程序已被激活后，要执行的委托调用，刚好与applicationDidEnterBackground 方法相对应。

6、applicationWillTerminate：在应用程序要完全推出的时候，要执行的委托调用，这个需要要设置UIApplicationExitsOnSuspend的键值。

初次启动： 

```objective-c
iOS_didFinishLaunchingWithOptions
iOS_applicationDidBecomeActive
```

按下home键： 

```
iOS_applicationWillResignActive
iOS_applicationDidEnterBackground
```

点击程序图标进入： 

```
iOS_applicationWillEnterForeground
iOS_applicationDidBecomeActive
```

当应用程序进入后台时,应该保存用户数据或状态信息，所有没写到磁盘的文件或信息，在进入后台时，最后都写到磁盘去，因为程序可能在后台被杀死。释放尽可能释放的内存。

```
- (void)applicationDidEnterBackground:(UIApplication *)application
```

方法有大概5秒的时间让你完成这些任务。如果超过时间还有未完成的任务，你的程序就会被终止而且从内存中清除。

如果还需要长时间的运行任务，可以在该方法中调用：

```
[application beginBackgroundTaskWithExpirationHandler:^{ 
    NSLog(@"begin Background Task With Expiration Handler"); 
}];
```

**程序终止**

程序只要符合以下情况之一，只要进入后台或挂起状态就会终止：

①iOS 4.0以前的系统

②app是基于iOS 4.0之前系统开发的。

③设备不支持多任务

④在Info.plist文件中，程序包含了 UIApplicationExitsOnSuspend 键。

系统常常是为其他app启动时由于内存不足而回收内存最后需要终止应用程序，但有时也会是由于app很长时间才响应而终止。如果app当时运行在后台并且没有暂停，系统会在应用程序终止之前调用app的代理的方法 - (void)applicationWillTerminate:(UIApplication *)application，这样可以让你可以做一些清理工作。你可以保存一些数据或app的状态。这个方法也有5秒钟的限制。超时后方法会返回程序从内存中清除。

注意：用户可以手工关闭应用程序。



### 4. 一工人给老板打7天工要求一块金条 这金条只能切2次 工人每天要1/7金条 怎么分?

这道题解决的主要难点在于：不是给出去的就收不回来了，可以用交换的方法。　　

把金条分成三段（就是分两次，或者切两刀），分别是整根金条的1/7、2/7、 4/7。　　

第一天：给1/7的， 第二天：给2/7的，收回1/7的； 第三天，给1/7的； 第四天：给4/7的，收回1/7和2/7的 ；第五天：给1/7的 ；第六天：给2/7的，收回1/7的；第七天发1/7。

### 5. iOS中socket使用

Socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，我们才能使用TCP/IP协议。

- http协议：对应于应用层
- tcp协议：对应于传输层
- ip协议：对应于网络层

三者本质上没有可比性。 何况HTTP协议是基于TCP连接的。

TCP/IP是传输层协议，主要解决数据如何在网络中传输；而HTTP是应用层协议，主要解决如何包装数据。

我 们在传输数据时，可以只使用传输层（TCP/IP），但是那样的话，由于没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用应用层 协议，应用层协议很多，有HTTP、FTP、TELNET等等，也可以自己定义应用层协议。WEB使用HTTP作传输层协议，以封装HTTP文本信息，然 后使用TCP/IP做传输层协议将它发送到网络上。

### SOCKET原理

**1、套接字（socket）概念**

套接字（socket）是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。

应 用层通过传输层进行数据通信时，TCP会遇到同时为多个应用程序进程提供并发服务的问题。多个TCP连接或多个应用程序进程可能需要通过同一个 TCP协议端口传输数据。为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP／IP协议交互提供了套接字(Socket)接口。应 用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务。

**2、建立socket连接**

建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket，另一个运行于服务器端，称为ServerSocket。

套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。

服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。

客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

连 接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户 端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

**3、SOCKET连接与TCP连接**

创建Socket连接时，可以指定使用的传输层协议，Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接。

**4、Socket连接与HTTP连接**

由 于通常情况下Socket连接就是TCP连接，因此Socket连接一旦建立，通信双方即可开始相互发送数据内容，直到双方连接断开。但在实际网络应用 中，客户端到服务器之间的通信往往需要穿越多个中间节点，例如路由器、网关、防火墙等，大部分防火墙默认会关闭长时间处于非活跃状态的连接而导致 Socket 连接断连，因此需要通过轮询告诉网络，该连接处于活跃状态。

而HTTP连接使用的是“请求—响应”的方式，不仅在请求时需要先建立连接，而且需要客户端向服务器发出请求后，服务器端才能回复数据。

很 多情况下，需要服务器端主动向客户端推送数据，保持客户端与服务器数据的实时与同步。此时若双方建立的是Socket连接，服务器就可以直接将数据传送给 客户端；若双方建立的是HTTP连接，则服务器需要等到客户端发送一次请求后才能将数据传回给客户端，因此，客户端定时向服务器端发送连接请求，不仅可以 保持在线，同时也是在“询问”服务器是否有新的数据，如果有就将数据传给客户端。

### 6. 网络请求中post和get的区别

GET是用于获取数据的，POST一般用于将数据发给服务器之用。

**普遍答案**

1.GET使用URL或Cookie传参。而POST将数据放在BODY中。

2.GET的URL会有长度上的限制，则POST的数据则可以非常大。

3.POST比GET安全，因为数据在地址栏上不可见。

### 7.  支付宝SDK使用 

使用支付宝进行一个完整的支付功能，大致有以下步骤：向支付宝申请, 与支付宝签约，获得商户ID（partner）和账号ID（seller）和私钥(privateKey)。下载支付宝SDK，生成订单信息,签名加密调用支付宝客户端，由支付宝客户端跟支付宝安全服务器打交道。支付完毕后,支付宝客户端会自动跳回到原来的应用程序，在原来的应用程序中显示支付结果给用户看。

集成之后可能遇到的问题

**1）集成SDK编译时找不到 openssl/asn1.h 文件**

![](https://ws3.sinaimg.cn/large/006tNc79ly1fvrm3e4ajdj30go04zq3m.jpg)



解决方案：Build Settings --> Search Paths --> Header Search paths : $(SRCROOT)/支付宝集成/Classes/Alipay

![](https://ws4.sinaimg.cn/large/006tNc79ly1fvrm3u22pbj30go07bjsa.jpg)

**2）链接时：找不到 SystemConfiguration.framework 这个库**

![](https://ws3.sinaimg.cn/large/006tNc79ly1fvrm444cfwj30go0apgo0.jpg)

解决方案：

![](https://ws1.sinaimg.cn/large/006tNc79ly1fvrm4cis02j309x04g0t5.jpg)

打开支付宝客户端进行支付(用户没有安装支付宝客户端,直接在应用程序中添加一个WebView,通过网页让用户进行支付)

```objective-c
// 注意:如果是通过网页支付完成,那么会回调该block:callback
[[AlipaySDK defaultService] payOrder:orderString fromScheme:@"jingdong" callback:^(NSDictionary *resultDic) { }];
```

在AppDelegate.m 

```objective-c
// 当通过别的应用程序,将该应用程序打开时,会调用该方法
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options{ // 当用户通过支付宝客户端进行支付时,会回调该block:standbyCallback 
[[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) { NSLog(@"result = %@",resultDic); }]; return YES;}
```

### 9. 远程推送

当服务端远程向APNS推送至一台离线的设备时，苹果服务器Qos组件会自动保留一份最新的通知，等设备上线后，Qos将把推送发送到目标设备上。

**远程推送的基本过程：**

1.客户端的app需要将用户的UDID和app的bundleID发送给apns服务器,进行注册,apns将加密后的device Token返回给app

2.app获得device Token后,上传到公司服务器

3.当需要推送通知时,公司服务器会将推送内容和device Token一起发给apns服务器

4.apns再将推送内容送到客户端上

**创建证书的流程：**

1.打开钥匙串，生成CertificateSigningRequest.certSigningRequest文件

2.将CertificateSigningRequest.certSigningRequest上传进developer，导出.cer文件

3.利用CSR导出P12文件

4.需要准备下设备token值（无空格）

5.使用OpenSSL合成服务器所使用的推送证书

**本地app代码参考**

1.注册远程通知

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions//中注册远程通知
{
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound)];
}
```

2,实现几个代理方法： 

```objective-c
//获取deviceToken令牌  
-(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken  
{  
    //获取设备的deviceToken唯一编号  
    NSLog(@"deviceToken=%@",deviceToken);  
    NSString *realDeviceToken=[NSString stringWithFormat:@"%@",deviceToken];  
    //去除<>  
    realDeviceToken = [realDeviceToken stringByReplacingOccurrencesOfString:@"<" withString:@""];  
    realDeviceToken = [realDeviceToken stringByReplacingOccurrencesOfString:@">" withString:@""];  
    NSLog(@"realDeviceToken=%@",realDeviceToken);  
    [[NSUserDefaults standardUserDefaults] setValue:realDeviceToken forKey:@"DeviceToken"];  //要发送给服务器
}  
 //获取令牌出错  
-(void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error  
{  
    //注册远程通知设备出错  
    NSLog(@"RegisterForRemoteNotification error=%@",error);  
}  
//在应用在前台时受到消息调用  
-(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo  
{  
   //打印推送的消息  
    NSLog(@"%@",[[userInfo objectForKey:@"aps"] objectForKey:@"alert"]):  
}
```

**配置后台模式**

一般我们是使用开发版本的Provisioning做推送测试,如果没有问题,再使用发布版本证书的时候一般也应该是没有问题的。为了以防万一,我们可以在越狱的手机上安装我们的使用发布版证书的ipa文件(最好使用debug版本,并打印出获取到的deviceToken),安装成功后在;XCode->Window->Organizer-找到对应的设备查看console找到打印的deviceToken。

在后台的推送程序中使用发布版制作的证书并使用该deviceToken做推送服务.

使用开发和发布证书获取到的deviceToken是不一样的。

### 10. @protocol 和 category 中如何使用 @property

1）在protocol中使用property只会生成setter和getter方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性

2）category 使用 @property 也是只会生成setter和getter方法的声明,如果我们真的需要给category增加属性的实现,需要借助于运行时的两个函数：

```
①objc_setAssociatedObject
②objc_getAssociatedObject
```











