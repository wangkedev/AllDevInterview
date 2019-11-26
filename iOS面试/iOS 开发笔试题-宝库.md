参考答案不唯一，大家可以根据自己的理解回答，没有必要跟笔者的一样。参考笔者的答案，也许给你带来灵感！

### 1、对数组中的元素去重复

例如：

NSArray *array = @[@"12-11", @"12-11", @"12-11", @"12-12", @"12-13", @"12-14"];

参考答案：

- 第一种方法：开辟新的内存空间，然后判断是否存在，若不存在则添加到数组中，得到最终结果的顺序不发生变化。效率分析：时间复杂度为O ( n2)：

  ```objective-c
  NSMutableArray *resultArray = [[NSMutableArray alloc] initWithCapacity:array.count];
  
  // 外层一个循环
  
  for  (NSString *item  in  array) {
  
      // 调用-containsObject:本质也是要循环去判断，因此本质上是双层遍历
  
      // 时间复杂度为O ( n^2 )而不是O (n)
  
       if  (![resultArray containsObject:item]) {
  
         [resultArray addObject:item];
  
       }
  
  }
  
  NSLog(@ "resultArray: %@" , resultArray);
  
  补充：原来集合操作可以通过valueForKeyPath来实现的，去重可以一行代码实现：
  
  array = [array valueForKeyPath:@ "@distinctUnionOfObjects.self" ];
  
  NSLog(@ "%@" , array);
  
  但是返回的结果是无序的，与原来的顺序不同。大家可以阅读：Collection Operators
  
  ```

  

- 第二种方法：利用NSDictionary去重，字典在设置key-value时，若已存在则更新值，若不存在则插入值，然后获取allValues。若不要求有序，则可以采用此种方法。若要求有序，还得进行排序。效率分析：只需要一个循环就可以完成放入字典，若不要求有序，时间复杂度为O(n)。若要求排序，则效率与排序算法有关：

  ```objective-c
  NSMutableDictionary *resultDict = [[NSMutableDictionary alloc] initWithCapacity:array.count];
  
  for  (NSString *item  in  array) {
  
       [resultDict setObject:item forKey:item];
  
  }
  
  NSArray *resultArray = resultDict.allValues;
  
  NSLog(@ "%@" , resultArray);
  
  如果需要按照原来的升序排序，可以这样：
  
  resultArray = [resultArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
  
     NSString *item1 = obj1;
  
     NSString *item2 = obj2;
  
     return  [item1 compare:item2 options:NSLiteralSearch];
  
  }];
  
  NSLog(@ "%@" , resultArray);
  
  ```

- 第三种方法：利用集合NSSet的特性(确定性、无序性、互异性)，放入集合就自动去重了。但是它与字典拥有同样的无序性，所得结果顺序不再与原来一样。如果不要求有序，使用此方法与字典的效率应该是差不多的。效率分析：时间复杂度为O (n)：

  ```objective-c
  NSSet *set = [NSSet setWithArray:array];
  
  NSArray *resultArray = [set allObjects];
  
  NSLog(@ "%@" , resultArray);
  
  如果要求有序，那就得排序，比如这里要升序排序：
  
  resultArray = [resultArray sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
  
     NSString *item1 = obj1;
  
     NSString *item2 = obj2;
  
     return  [item1 compare:item2 options:NSLiteralSearch];
  
  }];
  
  NSLog(@ "%@" , resultArray);
  
  补充：
  
  一直没有使用过有序集合，网友们反馈到可以直接使用有序集合，感谢大家：
  
  NSOrderedSet *set = [NSOrderedSet orderedSetWithArray:array];
  
  NSLog(@ "%@" , set.array);
  
  ```

  

### 2、说说以下元素的特性和作用

NSArray、NSSet、NSDictionary与NSMutableArray、NSMutableSet、NSMutableDictionary

参考答案：

特性：

- NSArray表示不可变数组，是有序元素集，只能存储对象类型，可通过索引直接访问元素，而且元素类型可以不一样，但是不能进行增、删、改操作；NSMutableArray是可变数组，能进行增、删、改操作。通过索引查询值很快，但是插入、删除等效率很低。
- NSSet表示不可变集合，具有确定性、互异性、无序性的特点，只能访问而不能修改集合；NSMutableSet表示可变集合，可以对集合进行增、删、改操作。集合通过值查询很快，插入、删除操作极快。
- NSDictionary表示不可变字典，具有无序性的特点，每个key对应的值是唯一的，可通过key直接获取值；NSMutableDictionary表示可变字典，能对字典进行增、删、改操作。通过key查询值、插入、删除值都很快。

作用：

- 数组用于处理一组有序的数据集，比如常用的列表的dataSource要求有序，可通过索引直接访问，效率高。
- 集合要求具有确定性、互异性、无序性，在iOS开发中是比较少使用到的，笔者也不清楚如何说明其作用
- 字典是键值对数据集，操作字典效率极高，时间复杂度为常量，但是值是无序的。在ios中，常见的JSON转字典，字典转模型就是其中一种应用。

### 3、简单描述一下XIB与Storyboards，说一下他们的优缺点。

参考答案：笔者倾向于纯代码开发，所以所提供的参考答案可能具有一定的个人感情，不过还是给大家说说笔者的想法。

优点：

- XIB：在编译前就提供了可视化界面，可以直接拖控件，也可以直接给控件添加约束，更直观一些，而且类文件中就少了创建控件的代码，确实简化不少，通常每个XIB对应一个类。
- Storyboard：在编译前提供了可视化界面，可拖控件，可加约束，在开发时比较直观，而且一个storyboard可以有很多的界面，每个界面对应一个类文件，通过storybard，可以直观地看出整个App的结构。

缺点：

- XIB：需求变动时，需要修改XIB很大，有时候甚至需要重新添加约束，导致开发周期变长。XIB载入相比纯代码自然要慢一些。对于比较复杂逻辑控制不同状态下显示不同内容时，使用XIB是比较困难的。当多人团队或者多团队开发时，如果XIB文件被发动，极易导致冲突，而且解决冲突相对要困难很多。
- Storyboard：需求变动时，需要修改storyboard上对应的界面的约束，与XIB一样可能要重新添加约束，或者添加约束会造成大量的冲突，尤其是多团队开发。对于复杂逻辑控制不同显示内容时，比较困难。当多人团队或者多团队开发时，大家会同时修改一个storyboard，导致大量冲突，解决起来相当困难。

### 4、请把字符串2015-04-10格式化日期转为NSDate类型

参考答案：

```objective-c
NSString *timeStr = @ "2015-04-10" ;

NSDateFormatter *formatter = [[NSDateFormatter alloc] init];

formatter.dateFormat = @ "yyyy-MM-dd" ;

formatter.timeZone = [NSTimeZone defaultTimeZone];

NSDate *date = [formatter dateFromString:timeStr];

// 2015-04-09 16:00:00 +0000

NSLog(@ "%@" , date);

```



### 5、在App中混合HTML5开发App如何实现的。在App中使用HTML5的优缺点是什么？

参考答案：

在iOS中，通常是通常UIWebView来实现，当然在iOS8以后可以使用WKWebView来实现.有以下几种实现方法：

通过实现UIWebView的代理方法来拦截，判断scheme是否是约定好的，然后iOS调用本地相关API来实现：

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;

在iOS7以后，可以直接通过JavaScripteCore这个库来实现，通过往JS DOM注入对象，而这个对象对应于我们iOS的某个类的实例。更详细请阅读：

- OC JavaScriptCore与js交互
- WKWebView新特性及JS交互
- Swift JavaScriptCore与JS交互

可以通过WebViewJavascriptBridge来实现。具体如何使用，请大家去其它博客搜索吧！

优缺点：

- iOS加入H5响应比原生要慢很多，体验不太好，这是缺点。
- iOS加入H5可以实现嵌入别的功能入口，可随时更改，不用更新版本就可以上线，这是最大的优点

### 6、请描述一下同步和异步，说说它们之间的区别。

参考答案：

首先，我们要明确一点，同步和异步都是在线程中使用的。在iOS开发中，比如网络请求数据时，若使用同步请求，则只有请求成功或者请求失败得到响应返回后，才能继续往下走，也就是才能访问其它资源（会阻塞了线程）。网络请求数据异步请求时，不会阻塞线程，在调用请求后，可以继续往下执行，而不用等请求有结果才能继续。

区别：

- 线程同步：是多个线程访问同一资源时，只有当前正在访问的线程访问结束之后，其他线程才能开始访问（被阻塞）。
- 线程异步：是多个线程在访问竞争资源时，可以在空闲等待时去访问其它资源（不被阻塞）。

### 7、请简单描述一下队列和多线程的使用原理。

参考答案：

在iOS中队列分为以下几种：

- 串行队列：队列中的任务只会顺序执行

dispatch_queue_t q = dispatch_queue_create("...", DISPATCH_QUEUE_SERIAL);

- 

- 并行队列： 队列中的任务通常会并发执行

- 全局队列：是系统的，直接拿过来（GET）用就可以；与并行队列类似

dispatch_queue_t q = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

- 主队列：每一个应用程序对应唯一一个主队列，直接GET即可；在多线程开发中，使用主队列更新UI

dispatch_queue_t q = dispatch_get_main_queue();

上面这四种是针对GCD来讲的，串行队列中的任务只能一个个地执行，在前一个没有执行完毕之前，下一个只能等待。并行队列可以并发地执行任务，因此多个任务之间执行的顺序不能确定，当添加一个新的任务时，交由GCD来判断是否要创建新的新的线程。

大家可以阅读图片多线程，也许更明了：iOS图解多线程

### 8、描述一下iOS的内存管理，在开发中对于内存的使用和优化包含哪些方面。我们在开发中应该注意哪些问题。

参考答案：

内存管理准则：谁强引用过，谁就在不再使用时使引用计数减一。

对于内存的使用和优化常见的有以下方面：

- 重用问题：如UITableViewCells、UICollectionViewCells、UITableViewHeaderFooterViews设置正确的reuseIdentifier，充分重用。
- 尽量把views设置为不透明：当opque为NO的时候，图层的半透明取决于图片和其本身合成的图层为结果，可提高性能。
- 不要使用太复杂的XIB/Storyboard：载入时就会将XIB/storyboard需要的所有资源，包括图片全部载入内存，即使未来很久才会使用。那些相比纯代码写的延迟加载，性能及内存就差了很多。
- 选择正确的数据结构：学会选择对业务场景最合适的数组结构是写出高效代码的基础。比如，数组: 有序的一组值。使用索引来查询很快，使用值查询很慢，插入/删除很慢。字典: 存储键值对，用键来查找比较快。集合: 无序的一组值，用值来查找很快，插入/删除很快。
- gzip/zip压缩：当从服务端下载相关附件时，可以通过gzip/zip压缩后再下载，使得内存更小，下载速度也更快。
- 延迟加载：对于不应该使用的数据，使用延迟加载方式。对于不需要马上显示的视图，使用延迟加载方式。比如，网络请求失败时显示的提示界面，可能一直都不会使用到，因此应该使用延迟加载。
- 数据缓存：对于cell的行高要缓存起来，使得reload数据时，效率也极高。而对于那些网络数据，不需要每次都请求的，应该缓存起来，可以写入数据库，也可以通过plist文件存储。
- 处理内存警告：一般在基类统一处理内存警告，将相关不用资源立即释放掉
- 重用大开销对象：一些objects的初始化很慢，比如NSDateFormatter和NSCalendar，但又不可避免地需要使用它们。通常是作为属性存储起来，防止反复创建。
- 避免反复处理数据：许多应用需要从服务器加载功能所需的常为JSON或者XML格式的数据。在服务器端和客户端使用相同的数据结构很重要。
- 使用Autorelease Pool：在某些循环创建临时变量处理数据时，自动释放池以保证能及时释放内存。
- 正确选择图片加载方式：详情阅读细读UIImage加载方式

### 9、plist文件是用来做什么的。一般用它来处理一些什么方面的问题。

参考答案：

plist是iOS系统中特有的文件格式。我们常用的NSUserDefaults偏好设置实质上就是plist文件操作。plist文件是用来持久化存储数据的。

我们通常使用它来存储偏好设置，以及那些少量的、数组结构比较复杂的不适合存储数据库的数据。比如，我们要存储全国城市名称和id，那么我们要优先选择plist直接持久化存储，因为更简单。

### 10、iOS中缓存一定量的数据以便下次可以快速执行，那么数据会存储在什么地方，有多少种存储方式？

参考答案：

- 偏好设置(NSUserDefaults)
- plist文件存储
- 归档
- SQLite3
- Core Data

详情请阅读：iOS常用的持久化存储方式

### 11、请简单写出增、删、改、查的SQL语句。

参考答案：

数据库的简单操作，还是会的，大学可没白学。

- 增：

insert into tb_blogs(name, url) values('标哥的技术博客','http://www.henishuo.com');

- 删：

delete from tb_blogs where blogid = 1;

- 改：

- update tb_blogs set url = 'www.henishuo.com' where blogid = 1;
- update tb_blogs set url = 'www.henishuo.com' where blogid = 1;

- 查：

select name, url from tb_blogs where blogid = 1;

### 12、在提交苹果审核时，遇到哪些问题被拒绝，对于被拒绝的问题是如何处理的。

参考答案：

对于笔者而言，所提交过的app还没有被拒绝过。不过在笔者所维护的几个群里经常有朋友们问到相关被拒绝的解决办法。幸好还懂一点点英文，还能帮助他们翻译翻译苹果反馈的被拒原因及所担的建议。

这里只列出几种最常见的被拒原因：

- 最常见到的就是app中有虚拟物品交易，但是不是走内购导致被拒。
- 音频类App或者使用到音频相关的app，因为版权问题而被拒
- App出现必闪退而被拒