### 目录

- 实际问题

- - 子线程同时执行ABC三个同步任务、全部执行完成再在子线程执行三个同步任务EDF。
  - 上一题中的ABC三个任务改成异步任务(如AFN网络请求)、全部回调成功后进行数据整合。
  - 实现本地大体量数组内容的实时输入搜索(如通讯录搜索好友名称/ID)。

- 问题汇总

- - 并行和并发的区别？
  - 串行/并行、同步异步的区别?(附带如何判断GCD的执行顺序、是否开辟线程)
  - NSOperation与GCD的关系
  - 默认最大并发
  - 线程取消
  - [thread cancel]可以关闭线程？
  - performSelector开头那么多方法为什么分散在不同的文件里？
  - NSBlockOperation和NSInvocationOperation有什么关系和区别。
  - NSInvocationOperation如何解决参数受限的问题
  - NSOperation可以像GCD一样设置串行并行么？
  - NSOperation队列内操作执行的时间点
  - NSOperation设置优先级是否可以直接决定操作的执行顺序?
  - NSBlockOperation用addExecutionBlock追加的操作、是否为串行执行。如果不是、为什么要这么设计？
  - 主队列([NSOperationQueue mainQueue])可以不可以修改最大并发数？主队列下添加的操作、都会在主线程执行么？
  - GCD的并行队列一定会开辟新的线程？
  - dispatch_once如何实现一次性代码？
  - NSOperation的添加进队列后可不可以追加依赖?GCD任务组添加监听后可不可以追加任务?

- 不同线程对比

  ---

  

#### 实际问题

这个不太可能让人真的手写代码或者上机、根据回答的点大概可以揣测一些水平吧？

- 子线程同时执行ABC三个同步任务、全部执行完成再在子线程执行三个同步任务EDF。

说队列组/依赖基本可以确定了解GCD/NSOpertion。但是比较麻烦、用线程栅栏dispatch_barrier的话会更简便一些。

- 上一题中的ABC三个任务改成异步任务(如AFN网络请求)、全部回调成功后进行数据整合。

如果只说队列/任务组肯定不行。因为网络请求本身是异步的、任务会立即完成、但数据还没有回来。

最好的就是在队列组的前提下。把异步的网络请求转化为同步、以捕获正确的完成时机。

具体操作需要使用信号量。

- 实现本地大体量数组内容的实时输入搜索(如通讯录搜索好友名称/ID)

1、每次输入字符的时候。如何进行数据遍历(如果用的For、那么为什么不用GCD的快速迭代？----因为开辟线程以及线程同步需要些许耗时、对于非耗时操作、for的性能会更好一些)。

2、每次输入字符的时候。如何废弃之前的搜索任务、以免重复插入(NSOperationQueue之类)。

3、高频次使用searchArray数组时的安全性。(锁)

#### 问题汇总

- 并行和并发的区别？

- - 并发是指两个或多个事件在同一时间间隔内发生。

例如单CPU的处理多线程。

- - 并行是指两个或者多个事件在同一时刻发生

例如多CPU的处理多线程。

- 串行/并行、同步异步的区别?(附带如何判断GCD的执行顺序、是否开辟线程)

- - 串行/并行针对队列。代表是否开辟新的线程去执行任务。
  - 同步/异步针对任务。代表是否阻塞当前线程。

队列为先决条件、然后再去满足任务的需求。

举个几个例子:

1、异步+串行(如果不是主队列)：

可以开辟线程 + 阻塞线程 = 会开辟一个新的线程去执行任务。

2、同步+并行：

不会开辟线程 + 不能阻塞线程 = 只能将任务放置与当前线程的最后、以解放线程。

2、异步+并行：

可以开辟线程 + 不能阻塞线程 = 开辟多个线程执行任务

- NSOperation与GCD的关系

GCD基于C、NSOperation基于GCD的封装。

- 默认最大并发

- - NSThread

本身并不会限制、也不支持限制最大并发(起码支持是四位数以内、如果超过某个阈值会error[NSThread start]: Thread creation failed with error 35)

```objective-c
- (void)thread_bingfa { for (int a = 0; a < 1000; a ++) {

        [self performSelectorInBackground:@selector(aaa:) withObject:[NSNumber numberWithInt:a]];

    }

}

- (void)aaa:(NSNumber *)number { for (int a = 0; a < 100; a++) {

        sleep(1); NSLog(@"%@",number);

    }

}

```



![](https://ws3.sinaimg.cn/large/006tNc79ly1fvrp5q163bj30jg07ddhg.jpg)

- NSOperation

默认的限制大概三位数以下(我模拟器分配到了63)

```objective-c
self.operationQueue=[[NSOperationQueue alloc]init];

- (void)viewDidLoad {
    for (int a = 0; a < 10000; a ++) {
        [self bbb:a];
    }
}

- (void)bbb:(int)a {
        [self.operationQueue addOperationWithBlock:^{
            for (int i = 0; i < 100; i ++) {
                sleep(10);
                NSLog(@"%d",a);
            }
        }];
}
```



![img](/Users/Ethan/Desktop/sinp裁图/未命名文件夹/846FA64901264498A7B5D2CB4DF91292.jpeg)

- GCD

默认的最大并发和NSOperation相同。

毕竟NSOperation基于GCD的OC封装、倒也说得通。

- [thread cancel]可以关闭线程？

不能、只能把对应线程进行cancel标记。详见下文NSThread相关的几个坑。

- performSelector开头那么多方法为什么分散在不同的文件里?

分散在NSThread.h、NSRunLoop.h、NSObject.h。

详见下文一些NSObject的相关扩展方法（performSelector）.

- NSBlockOperation和NSInvocationOperation有什么关系和区别。

- - 二者都是NSOperation的子类、都可以被添加进队列中(或者自己主动)执行。
  - NSBlockOperation可以解决NSInvocationOperation传递参数受限的问题。

- NSInvocationOperation如何解决参数受限的问题

这个问题其实和解决- (id)performSelector:(SEL)aSelector withObject:(id)object下方法受限的方式一样。

1、使用字典。

2、NSInvocationOperation实际上就是方法签名NSInvocation。所以如果使用NSInvocation进行初始化也能解决参数受限的问题、只是太麻烦了、除非特定情况(目前我接触到的只有模块化的Route层)不然不推荐。

- NSOperation可以像GCD一样设置串行并行么？

串行并行实际上是GCD的名词。

并行意味着多线程执行任务、串行意味着单线程执行任务。

任务在每一个线程内部、其实都是串行的。

NSOperation并没有串行并行的概念、自然也谈不上设置。

但是我们可以通过通过设置某个队列(NSOperationQueue)的大并发数为1、让其中任务们(NSOperation)自动被分配到不同线程中自动执行、以达到串行/并行的底层结果。

- NSOperation队列内操作执行的时间点:

- - 所有操作在被添加到队列中时、立即进行如下判断:

- - - 如果所插入的操作存在依赖关系、优先完成依赖操作。
    - 如果所插入的操作不存在依赖关系、队列并发数为1下采用先进先出的原则、反之直接开辟新的线程执行。

（具体可见下文NSOperation --> 操作的执行顺序）

- - 当一个操作执行完成之后、队列会取出对其有依赖的所有操作、进行下一步判断：

- - - 如果该操作没有其他依赖(准备就绪、isReady属性)、进行下一步判断
    - 所有可以执行的操作根据优先级排序执行。

（具体可见下文NSOperation -->  操作的优先级）

- NSOperation设置优先级是否可以直接决定操作的执行顺序?

不能、优先级的判定是建立在依赖操作完成后对下一步操作的排序下。

具体可见下文NSOperation --> 操作的优先级

- NSBlockOperation用addExecutionBlock追加的操作、是否为串行执行。如果不是、为什么要这么设计？

不是、默认的操作会被置于队列开辟的首个线程(主队列则为主线程)、剩余的操作会开辟新的线程并发执行。但是有并发数限制、由系统分配。

至于为什么这么设计。

NSBlockOperation下所有的操作默认情况下也是并行的。由并行通过依赖控制成串行容易、但由由串行想做出并行的效果则很难。

比如需要同时下载三张图片下载完成之后、将其展示。

我可以将三个下载操作追加进一个blockOperation1、再让展示操作的BlockOperation2依赖BlockOperation1。

```objective-c
NSOperationQueue *operationQueue=[NSOperationQueue mainQueue];

    NSBlockOperation *blockOperation1=[NSBlockOperation blockOperationWithBlock:^{
        sleep(1);
        NSLog(@"下载任务--%d",1);
    }];

    for (int i=2; i<4; ++i) {
        [blockOperation1 addExecutionBlock:^{
            sleep(1);
            NSLog(@"下载任务--%d",i);
        }];
    }

    NSBlockOperation *blockOperation2=[NSBlockOperation blockOperationWithBlock:^{
        sleep(1);
        NSLog(@"展示任务");
    }];

    [blockOperation2 addDependency:blockOperation1];

    [operationQueue addOperation:blockOperation1];

    [operationQueue addOperation:blockOperation2];

```



打印结果

```
2018-03-16 16:34:02.388806+0800 test[5620:522718] 下载任务--2

2018-03-16 16:34:02.388806+0800 test[5620:522602] 下载任务--1

2018-03-16 16:34:02.388806+0800 test[5620:522717] 下载任务--3

2018-03-16 16:34:03.390790+0800 test[5620:522602] 展示任务

```



如果addExecutionBlock的操作是串行的。那么我只能创建三个下载操作、然后将展示操作依赖于以上三个操作。得不偿失。

- 主队列([NSOperationQueue mainQueue])可以不可以修改最大并发数？主队列下添加的操作、都会在主线程执行么？

- - 不能、主队列的最大并发数始终为1(自定义队列默认为-1)、且修改无效。
  - 默认状况下是的、但也有例外（追加操作addExecutionBlock）。

- GCD的并行队列一定会开辟新的线程？

不

![](https://ws1.sinaimg.cn/large/006tNc79ly1fvrp8dviemj30jg090jrr.jpg)

GCD

- dispatch_once如何实现一次性代码？
- NSOperation的添加进队列后可不可以追加依赖?GCD任务组添加监听后可不可以追加任务?

- - NSOperation的依赖必须在添加进队列(并且执行前)之前设置。(但是我们可以对某被依赖的操作进行追加addExecutionBlock以延缓调用)
  - GCD任务组则具备追加任务的功能。前提是监听并未被触发。

不同线程对比

主要说GCD和NSOperation、如果NSThread方便实现的话可能会提一句。

线程切换

- NSThread

  ```objective-c
  - (void)viewDidLoad {
      [super viewDidLoad];
      // Do any additional setup after loading the view, typically from a nib.
      [self performSelectorInBackground:@selector(fun1) withObject:nil];
  }
  
  - (void)fun1 {
      //回到主线程
      [self performSelectorOnMainThread:@selector(fun2) withObject:nil waitUntilDone:nil];
  }
  
  ```

- GCD

创建/获取一个并行队列添加任务、然后返回主队列

```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)

dispatch_async(queue, ^{
    // 执行耗时的异步操作...
    dispatch_async(dispatch_get_main_queue(), ^{
      // 回到主线程，执行UI刷新操作
    });
});
```



- NSOperation

创建队列添加操作、然后返回主队列

```objective-c
NSOperationQueue * queue = [[NSOperationQueue alloc]init];
//切换到子线程
[queue addOperationWithBlock:^{
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        //切换回主线程
    }];
}];

```

- 队列组/依赖

- GCD

  ```objective-c
  - (void)dispatch_group_test {
      dispatch_queue_t queue = dispatch_queue_create("queue_test", DISPATCH_QUEUE_CONCURRENT);
      dispatch_group_t group = dispatch_group_create();
      dispatch_group_async(group, queue, ^{
          NSLog(@"任务1——准备休眠3秒");
          sleep(3);
          NSLog(@"任务1——完成");
      });
      NSLog(@"主线程——准备休眠5秒");
      sleep(5);
      NSLog(@"主线休眠结束");
  
      dispatch_group_async(group, queue, ^{
          NSLog(@"任务2——准备休眠10秒");
          sleep(10);
          NSLog(@"任务2——完成");
      });
  
      dispatch_group_notify(group, queue, ^{
          NSLog(@"任务组完成");
      });
      NSLog(@"主线程结束");
  
  }
  
  ```

- NSOperation

  ```
  //创建操作队列
  NSOperationQueue *operationQueue=[[NSOperationQueue alloc]init];
  
  //创建最后一个操作
  NSBlockOperation *lastBlockOperation=[NSBlockOperation blockOperationWithBlock:^{
      sleep(1);
      NSLog(@"最后的任务");
  }];
  
  for (int i=0; i<5-1; ++i) {
      //创建多线程操作
      NSBlockOperation *blockOperation=[NSBlockOperation blockOperationWithBlock:^{
          sleep(i);
          NSLog(@"第%d个任务",i);
      }];
  
      //设置依赖操作为最后一个操作
      [blockOperation addDependency:lastBlockOperation];
      [operationQueue addOperation:blockOperation];
  
  }
  
  //将最后一个操作加入线程队列
  [operationQueue addOperation:lastBlockOperation];
  ```

  

- GCD

- - 适用于多个任务同时执行、可以捕获所有任务完成的回调。
  - 任务添加到队列组、且未全部完成时可以向任务组中添加任务。

- NSOperation

- - 适用于多个任务之间相互依赖等待、最后完成的时候并没有回调。
  - 添加到队列中(并且已经执行)的操作不能再新增依赖、但是可以向追加操作。

串行队列

- GCD

  ```objective-c
  两种方式获取/创建
  //主队列--串行
  dispatch_queue_t queue1 = dispatch_get_main_queue();
  
  //自定义串行队列
  dispatch_queue_t queue2 = dispatch_queue_create("test_queue", DISPATCH_QUEUE_SERIAL);
  ```

  

- NSOperation

两种方式获取/创建

```
//主队列

NSOperationQueue * queue1 = [NSOperationQueue mainQueue];

//自定义队列 -- 把并发改为1

NSOperationQueue * queue2 = [[NSOperationQueue alloc]init];

queue2.maxConcurrentOperationCount = 1;

```



最大并发

- GCD

通过信号量进行约束。详见GCD-->信号量

```objective-c
// 创建队列组
dispatch_group_t group = dispatch_group_create();
// 创建信号量，并且设置值为3
dispatch_semaphore_t semaphore = dispatch_semaphore_create(3);
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

for (int i = 0; i < 100; i++){

    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    dispatch_group_async(group, queue, ^{
        NSLog(@"%i",i);
        sleep(2);
        // 每次发送信号则semaphore会+1，
        dispatch_semaphore_signal(semaphore);
    });
}

```

