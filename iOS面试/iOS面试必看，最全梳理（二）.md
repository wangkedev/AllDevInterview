# iOS面试必看，最全梳理（二）

### HTTP协议的特点，关于HTTP请求GET和POST的区别

GET和POST的区别：

- HTTP超文本传输协议，是短连接，是客户端主动发送请求，服务器做出响应，服务器响应之后，链接断开。HTTP是一个属于应用层面向对象的协议，HTTP有两类报文：请求报文和响应报文。
- HTTP请求报文：一个HTTP请求报文由请求行、请求头部、空行和请求数据4部分组成。
- HTTP响应报文：由三部分组成：状态行、消息报头、响应正文。
- GET请求：参数在地址后拼接，没有请求数据，不安全（因为所有参数都拼接在地址后面），不适合传输大量数据（长度有限制，为1024个字节）。

GET提交、请求的数据会附在URL之后，即把数据放置在HTTP协议头 中。

以？分割URL和传输数据，多个参数用&连接。如果数据是英文字母或数字，原样发送，如果是空格，转换为+，如果是中文/其他字符，则直接把字符串用BASE64加密。

- POST请求：参数在请求数据区放着，相对GET请求更安全，并且数据大小没有限制。把提交的数据放置在HTTP包的包体 中.
- GET提交的数据会在地址栏显示出来，而POST提交，地址栏不会改变。

传输数据的大小：

- GET提交时，传输数据就会受到URL长度限制，POST由于不是通过URL传值，理论上书不受限。

安全性：

- POST的安全性要比GET的安全性高；
- 通过GET提交数据，用户名和密码将明文出现在URL上，比如登陆界面有可能被浏览器缓存。
- HTTPS：安全超文本传输协议（Secure Hypertext Transfer Protocol），它是一个安全通信通道，基于HTTP开发，用于客户计算机和服务器之间交换信息，使用安全套结字层（SSI）进行信息交换，即HTTP的安全版。

### ASIHttpRequest、AFNetWorking之间的区别

- ASIHttpRequest功能强大，主要是在MRC下实现的，是对系统CFNetwork API进行了封装，支持HTTP协议的CFHTTP，配置比较复杂，并且ASIHttpRequest框架默认不会帮你监听网络改变，如果需要让ASIHttpRequest帮你监听网络状态改变，并且手动开始这个功能。
- AFNetWorking构建于NSURLConnection、NSOperation以及其他熟悉的Foundation技术之上。拥有良好的架构，丰富的API及模块构建方式，使用起来非常轻松。它基于NSOperation封装的，AFURLConnectionOperation子类。
- ASIHttpRequest是直接操作对象ASIHttpRequest是一个实现了NSCoding协议的NSOperation子类；AFNetWorking直接操作对象的AFHttpClient，是一个实现NSCoding和NSCopying协议的NSObject子类。
- 同步请求：ASIHttpRequest直接通过调用一个startSynchronous方法；AFNetWorking默认没有封装同步请求，如果开发者需要使用同步请求，则需要重写getPath:paraments:success:failures方法，对于AFHttpRequestOperation进行同步处理。
- 性能对比：AFNetworking请求优于ASIHttpRequest；

### XML数据解析方式各有什么不同，JSON解析有哪些框架？

- XML数据解析的两种解析方式：DOM解析和SAX解析；
- DOM解析必须完成DOM树的构造，在处理规模较大的XML文档时就很耗内存，占用资源较多，读入整个XML文档并构建一个驻留内存的树结构（节点树），通过遍历树结构可以检索任意XML节点，读取它的属性和值，通常情况下，可以借助XPath查询XML节点；
- SAX与DOM不同，它是事件驱动模型，解析XML文档时每遇到一个开始或者结束标签、属性或者一条指令时，程序就产生一个事件进行相应的处理，一边读取XML文档一边处理，不必等整个文档加载完才采取措施，当在读取解析过程中遇到需要处理的对象，会发出通知进行处理。因此，SAX相对于DOM来说更适合操作大的XML文档。
- JSON解析：性能比较好的主要是第三方的JSONKIT和iOS自带的JSON解析类，其中自带的JSON解析性能最高，但只能用于iOS5之后。

### 如何进行真机调试？

1.首先需要用钥匙串创建一个钥匙（key）；

2.将钥匙串上传到官网，获取iOS Development证书；

3.创建App ID即我们应用程序中的Boundle ID；

4.添加Device ID即UDID；

5.通过勾选前面所创建的证书：App ID、Device ID；

6.生成mobileprovision文件；

7.先决条件：申请开发者账号 99美刀

### APP发布的上架流程

1.登录[应用发布网站](https://itunesconnect.apple.com/)添加应用信息；

2.下载安装发布证书；

3.选择发布证书，使用Archive编译发布包，用Xcode将代码（发布包）上传到服务器；

4.等待审核通过;

5.生成IPA：菜单栏->Product->Archive.

### SVN的使用

- SVN=版本控制+备份服务器，可以把SVN当成备份服务器，并且可以帮助你记住每次上服务器的档案内容，并自动赋予每次变更的版本；
- SVN的版本控制：所有上传版本都会帮您记录下来，也有版本分支及合并等功能。SVN可以让不同的开发者存取同样的档案，并且利用SVN Server作为档案同步的机制，即您有档案更新时，无需将档案寄送给您的开发成员。SVN的存放档案方式是采用差异备份的方式，即会备份到不同的地方，节省硬盘空间，也可以对非文字文件进行差异备份。
- SVN的重要性：备份工作档案的重要性、版本控管的重要性、伙伴间的数据同步的重要性、备份不同版本是很耗费硬盘空间的；
- 防止冲突：

- 

1.防止代码冲突：不要多人同时修改同一文件，例如：A、B都修改同一个文件，先让A修改，然后提交到服务器，然后B更新下来，再进行修改；

2.服务器上的项目文件Xcodeproj，仅让一个人管理提交，其他人只更新，防止文件发生冲突。

### 如何进行网络消息推送

- 一种是Apple自己提供的通知服务（APNS服务器）、一种是用第三方推送机制。
- 首先应用发送通知，系统弹出提示框询问用户是否允许，当用户允许后向苹果服务器(APNS)请求deviceToken，并由苹果服务器发送给自己的应用，自己的应用将DeviceToken发送自己的服务器，自己服务器想要发送网络推送时将deviceToken以及想要推送的信息发送给苹果服务器，苹果服务器将信息发送给应用。
- 推送信息内容，总容量不超过256个字节；
- iOS SDK本身提供的APNS服务器推送，它可以直接推送给目标用户并根据您的方式弹出提示。

优点：不论应用是否开启，都会发送到手机端；

缺点：消息推送机制是苹果服务端控制，个别时候可能会有延迟，因为苹果服务器也有队列来处理所有的消息请求；

- 第三方推送机制，普遍使用Socket机制来实现，几乎可以达到即时的发送到目标用户手机端，适用于即时通讯类应用。

优点：实时的，取决于心跳包的节奏；

缺点：iOS系统的限制，应用不能长时间的后台运行，所以应用关闭的情况下这种推送机制不可用。

### 网络七层协议

- 应用层：

- 

1.用户接口、应用程序；

2.Application典型设备：网关；

3.典型协议、标准和应用：TELNET、FTP、HTTP

- 

- 表示层：

- 

1.数据表示、压缩和加密presentation

2.典型设备：网关

3.典型协议、标准和应用：ASCLL、PICT、TIFF、JPEG|MPEG

4.表示层相当于一个东西的表示，表示的一些协议，比如图片、声音和视频MPEG。

- 

- 会话层：

- 

1.会话的建立和结束；

2.典型设备：网关；

3.典型协议、标准和应用：RPC、SQL、NFS、X WINDOWS、ASP

- 

- 传输层：

- 

1.主要功能：端到端控制Transport；

2.典型设备：网关；

3.典型协议、标准和应用：TCP、UDP、SPX

- 

- 网络层：

- 

1.主要功能：路由、寻址Network；

2.典型设备：路由器；

3.典型协议、标准和应用：IP、IPX、APPLETALK、ICMP；

- 

- 数据链路层：

- 

1.主要功能：保证无差错的疏忽链路的data link；

2.典型设备：交换机、网桥、网卡；

3.典型协议、标准和应用：802.2、802.3ATM、HDLC、FRAME RELAY；

- 

- 物理层：

- 

1.主要功能：传输比特流Physical；

2.典型设备：集线器、中继器

3.典型协议、标准和应用：V.35、EIA/TIA-232.

### 对NSUserDefaults的理解

- NSUserDefaults：系统提供的一种存储数据的方式，主要用于保存少量的数据，默认存储到library下的Preferences文件夹。

### SDWebImage原理

调用类别的方法：

- 从内存中（字典）找图片（当这个图片在本次程序加载过），找到直接使用；
- 从沙盒中找，找到直接使用，缓存到内存。
- 从网络上获取，使用，缓存到内存，缓存到沙盒。

### OC中是否有二维数组，如何实现二维数组？

- OC中没有二维数组，可通过嵌套数组实现二维数组。

### LayoutSubViews在什么时候被调用？

- 当View本身的frame改变时，会调用这个方法。

### 深拷贝和浅拷贝

- 如果对象有个指针型成员变量指向内存中的某个资源，那么如何复制这个对象呢？你会只是复制指针的值传给副本的新对象吗？指针只是存储内存中资源地址的占位符。在复制操作中，如果只是将指针复制给新对象，那么底层的资源实际上仍然由两个实例在共享。

![](https://ws3.sinaimg.cn/large/006tNc79ly1fvrow8su14j30fh08q74g.jpg)

示例图1

- 浅复制：两个实例的指针仍指向内存中的同一资源，只复制指针值而不是实际资源；
- 深复制：不仅复制指针值，还复制指向指针所指向的资源。如下图：

![](https://ws3.sinaimg.cn/large/006tNc79ly1fvrowpn9zcj30du07eaa1.jpg)

示例图2

### 单例模式理解与使用

- 单例模式是一种常用设计模式，单例模式是一个类在系统中只有一个实例对象。通过全局的一个入口点对这个实例对象进行访问；
- iOS中单例模式的实现方式一般分为两种：非ARC和ARC+GCD。

### 对沙盒的理解

- 每个iOS应用都被限制在“沙盒”中，沙盒相当于一个加了仅主人可见权限的文件夹，及时在应用程序安装过程中，系统为每个单独的应用程序生成它的主目录和一些关键的子目录。苹果对沙盒有几条限制:

- 

1.应用程序在自己的沙盒中运作，但是不能访问任何其他应用程序的沙盒；

2.应用之间不能共享数据，沙盒里的文件不能被复制到其他应用程序的文件夹中，也不能把其他应用文件夹复制到沙盒中；

3.苹果禁止任何读写沙盒以外的文件，禁止应用程序将内容写到沙盒以外的文件夹中；

4.沙盒目录里有三个文件夹：Documents——存储；应用程序的数据文件，存储用户数据或其他定期备份的信息；Library下有两个文件夹，Caches存储应用程序再次启动所需的信息，Preferences包含应用程序的偏好设置文件，不可在这更改偏好设置；temp存放临时文件即应用程序再次启动不需要的文件。

- 获取沙盒根目录的方法，有几种方法：用NSHomeDirectory获取。
- 获取Document路径：NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES).

### 对瀑布流的理解

- 首先图片的宽度都是一样的，1.将图片等比例压缩，让图片不变形；2.计算图片最低应该摆放的位置，哪一列低就放在哪；3.进行最优排列，在ScrollView的基础上添加两个tableView，然后将之前所计算的scrollView的高度通过tableView展示出来。
- 如何使用两个TableView产生联动：将两个tableView的滚动事件禁止掉，最外层scrollView滚动时将两个TableView跟着滚动，并且更改contentOffset，这样产生效果滚动的两个tableView。

### ViewController 的 loadView、viewDidLoad、viewDidUnload 分别是在什么时候调用的？

- viewDidLoad在view从nib文件初始化时调用，loadView在controller的view为nil时调用。
- 此方法在编程实现view时调用,view控制器默认会注册memory warning notification,当view controller的任何view没有用的时候，viewDidUnload会被调用，在这里实现将retain的view release,如果是retain的IBOutlet view 属性则不要在这里release,IBOutlet会负责release 。

### 关键字volatile有什么含意?并给出三个不同的例子：

- 一个定义为volatile的变量是说这变量可能会被意想不到地改变，这样，编译器就不会去假设这个变量的值了。精确地说就是，优化器在用到这个变量时必须每次都小心地重新读取这个变量的值，而不是使用保存在寄存器里的备份。下面是volatile变量的几个例子：

- 

1.并行设备的硬件寄存器（如：状态寄存器）；

2.一个中断服务子程序中会访问到的非自动变量(Non-automatic variables)；

3.多线程应用中被几个任务共享的变量。

### @synthesize、@dynamic的理解

- @synthesize是系统自动生成getter和setter属性声明;@synthesize的意思是，除非开发人员已经做了，否则由编译器生成相应的代码，以满足属性声明；
- @dynamic是开发者自已提供相应的属性声明,@dynamic意思是由开发人员提供相应的代码：对于只读属性需要提供setter，对于读写属性需要提供 setter 和getter。查阅了一些资料确定@dynamic的意思是告诉编译器,属性的获取与赋值方法由用户自己实现, 不自动生成。

### frame和bounds有什么不同？

- frame指的是：该view在父view坐标系统中的位置和大小。（参照点是父亲的坐标系统）
- bounds指的是：该view在本身坐标系统中的位置和大小。（参照点是本身坐标系统）

### view的touch事件有哪些？

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;

- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;

- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event; 

- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;

### 自定义实现UITabbarController的原理

- 运用字典，点击五个按钮的一个可以从字典里选择一个控制器对象，将其View显示到主控制器视图上。

### iOS中的响应者链的工作原理

- 每一个应用有一个响应者链，我们的视图结构是一个N叉树(一个视图可以有多个子视图，一个子视图同一时刻只有一个父视图),而每一个继承UIResponder的对象都可以在这个N叉树中扮演一个节点。
- 当叶节点成为最高响应者的时候，从这个叶节点开始往其父节点开始追朔出一条链，那么对于这一个叶节点来讲，这一条链就是当前的响应者链。响应者链将系统捕获到的UIEvent与UITouch从叶节点开始层层向下分发，期间可以选择停止分发，也可以选择继续向下分发。
- 如需了解更多细节，请读[这篇文章](http://www.tuicool.com/articles/6VFn2q)。

### View和View之间传值方式

- 对象的property属性传值；
- 方法参数传值；
- NSUserDefault传值；
- 块传值。

### property属性的修饰符的作用

- getter=getName、setter=setName：设置setter与getter的方法名；
- readwrite、readonly：设置可供访问级别；
- assign：方法直接赋值，不进行任何retain操作，为了解决原类型与环循引用问题；
- retain：其setter方法对参数进行release旧值再retain新值，所有实现都是这个顺序；
- copy：其setter方法进行copy操作，与retain处理流程一样，先对旧值release，再copy出新的对象，retainCount为1。这是为了减少对上下文的依赖而引入的机制。
- nonatomic：非原子性访问，不加同步， 多线程并发访问会提高性能。注意，如果不加此属性，则默认是两个访问方法都为原子型事务访问。

### 对于Run Loop的理解

- RunLoop，是多线程的法宝，即一个线程一次只能执行一个任务，执行完任务后就会退出线程。主线程执行完即时任务时会继续等待接收事件而不退出。非主线程通常来说就是为了执行某一任务的，执行完毕就需要归还资源，因此默认是不运行RunLoop的；
- 每一个线程都有其对应的RunLoop，只是默认只有主线程的RunLoop是启动的，其它子线程的RunLoop默认是不启动的，若要启动则需要手动启动；
- 在一个单独的线程中，如果需要在处理完某个任务后不退出，继续等待接收事件，则需要启用RunLoop；
- NSRunLoop提供了一个添加NSTimer的方法，可以指定Mode，如果要让任何情况下都回调，则需要设置Mode为Common模式；
- 实质上，对于子线程的runloop默认是不存在的，因为苹果采用了懒加载的方式。如果我们没有手动调用[NSRunLoop currentRunLoop]的话，就不会去查询是否存在当前线程的RunLoop，也就不会去加载，更不会创建。

### SQLite中常用的SQL语句

- 创建表：creat table 表名 (字段名 字段数据类型 是否为主键, 字段名 字段数据类型, 字段名 字段数据类型...)；
- 增: insert into 表名 (字段1, 字段2...) values (值1, 值2...)；
- 删: delete from 表名 where 字段 = 值；

### XIB与Storyboards的优缺点

优点：

- XIB：在编译前就提供了可视化界面，可以直接拖控件，也可以直接给控件添加约束，更直观一些，而且类文件中就少了创建控件的代码，确实简化不少，通常每个XIB对应一个类。
- Storyboard：在编译前提供了可视化界面，可拖控件，可加约束，在开发时比较直观，而且一个storyboard可以有很多的界面，每个界面对应一个类文件，通过storybard，可以直观地看出整个App的结构。

缺点：

- XIB：需求变动时，需要修改XIB很大，有时候甚至需要重新添加约束，导致开发周期变长。XIB载入相比纯代码自然要慢一些。对于比较复杂逻辑控制不同状态下显示不同内容时，使用XIB是比较困难的。当多人团队或者多团队开发时，如果XIB文件被发动，极易导致冲突，而且解决冲突相对要困难很多。
- Storyboard：需求变动时，需要修改storyboard上对应的界面的约束，与XIB一样可能要重新添加约束，或者添加约束会造成大量的冲突，尤其是多团队开发。对于复杂逻辑控制不同显示内容时，比较困难。当多人团队或者多团队开发时，大家会同时修改一个storyboard，导致大量冲突，解决起来相当困难。

### 将字符串“2015-04-10”格式化日期转为NSDate类型

```objective-c
NSString *timeStr = @ "2015-04-10" ;

NSDateFormatter *formatter = [[NSDateFormatter alloc] init];

formatter.dateFormat = @ "yyyy-MM-dd" ;

formatter.timeZone = [NSTimeZone defaultTimeZone];

NSDate *date = [formatter dateFromString:timeStr];

// 2015-04-09 16:00:00 +0000

NSLog(@ "%@" , date);

```



### 队列和多线程的使用原理

在iOS中队列分为以下几种：

- 串行队列：队列中的任务只会顺序执行；

dispatch_queue_t q = dispatch_queue_create("...", DISPATCH_QUEUE_SERIAL);

- 并行队列： 队列中的任务通常会并发执行；

dispatch_queue_t q = dispatch_queue_create("......",DISPATCH_QUEUE_CONCURRENT);

- 全局队列：是系统的，直接拿过来（GET）用就可以；与并行队列类似；

dispatch_queue_t q = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

- 主队列：每一个应用程序对应唯一主队列，直接GET即可；在多线程开发中，使用主队列更新UI；

dispatch_queue_t q = dispatch_get_main_queue();

更多细节见下图：

![](https://ws2.sinaimg.cn/large/006tNc79ly1fvroyvzrubj30ya2h1wqn.jpg)

### 内存的使用和优化的注意事项

- 重用问题：如UITableViewCells、UICollectionViewCells、UITableViewHeaderFooterViews设置正确的reuseIdentifier，充分重用；
- 尽量把views设置为不透明：当opque为NO的时候，图层的半透明取决于图片和其本身合成的图层为结果，可提高性能；
- 不要使用太复杂的XIB/Storyboard：载入时就会将XIB/storyboard需要的所有资源，包括图片全部载入内存，即使未来很久才会使用。那些相比纯代码写的延迟加载，性能及内存就差了很多；
- 选择正确的数据结构：学会选择对业务场景最合适的数组结构是写出高效代码的基础。比如，数组: 有序的一组值。使用索引来查询很快，使用值查询很慢，插入/删除很慢。字典: 存储键值对，用键来查找比较快。集合: 无序的一组值，用值来查找很快，插入/删除很快。
- gzip/zip压缩：当从服务端下载相关附件时，可以通过gzip/zip压缩后再下载，使得内存更小，下载速度也更快。
- 延迟加载：对于不应该使用的数据，使用延迟加载方式。对于不需要马上显示的视图，使用延迟加载方式。比如，网络请求失败时显示的提示界面，可能一直都不会使用到，因此应该使用延迟加载。
- 数据缓存：对于cell的行高要缓存起来，使得reload数据时，效率也极高。而对于那些网络数据，不需要每次都请求的，应该缓存起来，可以写入数据库，也可以通过plist文件存储。
- 处理内存警告：一般在基类统一处理内存警告，将相关不用资源立即释放掉
- 重用大开销对象：一些objects的初始化很慢，比如NSDateFormatter和NSCalendar，但又不可避免地需要使用它们。通常是作为属性存储起来，防止反复创建。
- 避免反复处理数据：许多应用需要从服务器加载功能所需的常为JSON或者XML格式的数据。在服务器端和客户端使用相同的数据结构很重要;
- 使用Autorelease Pool：在某些循环创建临时变量处理数据时，自动释放池以保证能及时释放内存;
- 正确选择图片加载方式：详情阅读细读[UIImage加载方式](http://www.henishuo.com/image-loaded-method/)

### UIViewController的完整生命周期

```objective-c
-[ViewController initWithNibName:bundle:]；

-[ViewController init]；

-[ViewController loadView]；

-[ViewController viewDidLoad]；

-[ViewController viewWillDisappear:]；

-[ViewController viewWillAppear:]；

-[ViewController viewDidAppear:]；

-[ViewController viewDidDisappear:]；

```



### UIImageView添加圆角

- 最直接的方法就是使用如下属性设置：

**这是离屏渲染（off-screen-rendering），消耗性能的**

- 给UIImage添加生成圆角图片的扩展API：这是on-screen-rendering

```objective-c
- (UIImage *)imageWithCornerRadius:(CGFloat)radius {

    CGRect rect = (CGRect){0.f, 0.f, self.size};

    UIGraphicsBeginImageContextWithOptions(self.size, NO, UIScreen.mainScreen.scale);

    CGContextAddPath(UIGraphicsGetCurrentContext(),

      [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:radius].CGPath);

    CGContextClip(UIGraphicsGetCurrentContext());

    [self drawInRect:rect];

    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();

    UIGraphicsEndImageContext();

    return  image;

}

```

