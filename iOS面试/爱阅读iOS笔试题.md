# 爱阅读iOS笔试题

### 1、写出float x 与“零值”比较的if语句

const float EPSINON = 0.00001;

if ((x >= - EPSINON) && (x <= EPSINON)

不可将浮点变量用“==”或“！=”与数字比较，应该设法转化成“>=”或“<=”此类形式。

### 2、写出至少3种交换两个整型数的方式

（1）临时变量

（2）加法

（3）异或运算

### 3、用变量a给出下面的定义

（1）一个整型数

（2）一个指向整型数的指针

（3）一个指向指针的指针，它指向的指针是指向一个整型数

（4）一个有10个整型数的数组

（5）一个有10个指针的数组，该指针是指向一个整型数的

（6）一个指向有10个整型数数组的指针

（7）一个指向函数的指针，该函数有一个整型参数并返回整型

（8）一个有10个指针的数组，该指针指向一个函数，该函数返回整型

a) int a; // An integer

b) int *a; // A pointer to an integer

c) int **a; // A pointer to a pointer to an integer

d) int a[10]; // An array of 10 integers

e) int *a[10]; // An array of 10 pointers to integers

f) int (*a)[10]; // A pointer to an array of 10 integers

g) int (*a)(int); // A pointer to a function a that takes an integer argument and returns an integer

h) int (*a[10])(int); // An array of 10 pointers to functions that take an integer argument and return an integer

### 4、请写出下面代码在32位平台上的运行结果

int main(void)

{

char a[30];

char *b = (char *)malloc(20 * sizeof(char));

printf("%d\n", sizeof(a));

printf("%d\n", sizeof(b));

printf("%d\n", sizeof(a[3]));

printf("%d\n", sizeof(b+3));

printf("%d\n", sizeof(*(b+4)));

return 0 ;

}

30/4/1/4/1

### 5、求下面函数的返回值

int func(x){

int countx = 0;

while(x){

countx ++;

x = x & (x - 1);

}

return countx;

}

假定x = 9999；

返回值countx = 9999；

### 6、在一个对象的方法里面:self.name = “object”;和name ＝”object”有什么不同吗?

self.name = “object”会调用对象的setName()方法，

name = “object”会直接把object赋值给当前对象的name 属性。

并且 self.name 这样retainCount会加１,而name就不会。

### 7、Object-C有多继承吗？没有的话用什么代替？

没有， cocoa 中所有的类都是NSObject 的子类 多继承在这里是用protocol 委托代理 来实现的

OOD（Object-Oriented Design）的多态特性 在 obj-c 中通过委托来实现。

### 8、Objective-C 中 #import 和 #include 的区别

预编译指令

Objective-C：#import

C，C++：#include

\#import由gcc编译器支持

在 Objective-C 中，#import 被当成 #include 指令的改良版本来使用。除此之外，#import 确定一个文件只能被导入一次，这使你在递归包含中不会出现问题。#import比起#include的好处就是不会引起交叉编译

### 9、delegate和Notification的区别

delegate针对one-to-one关系，并且reciever可以返回值给sender；

notification 可以针对one-to-one/many/none,reciever无法返回值给sender；

所以，delegate用于sender希望接受到reciever的某个功能反馈值，notification用于通知多个object某个事件

### 10、为什么很多内置的类，如TableViewController的delegate的属性是assign不是retain？

循环引用

所有的引用计数系统，都存在循环应用的问题。例如下面的引用关系：

对象a创建并引用到了对象b.

对象b创建并引用到了对象c.

对象c创建并引用到了对象b.

这时候b和c的引用计数分别是2和1。当a不再使用b，调用release释放对b的所有权，因为c还引用了b，所以b的引用计数为1，b不会被释放。b不释放，c的引用计数就是1，c也不会被释放。从此，b和c永远留在内存中。

这种情况，必须打断循环引用，通过其他规则来维护引用关系。比如，我们常见的delegate往往是assign方式的属性而不是retain方式的属 性，赋值不会增加引用计数，就是为了防止delegation两端产生不必要的循环引用。如果一个UITableViewController 对象a通 过retain获取了UITableView对象b的所有权，这个UITableView对象b的delegate又是a，如果这个delegate是 retain方式的，那基本上就没有机会释放这两个对象了。

### 11、Object C中创建线程的方法是什么？如果在主线程中执行代码，方法是什么？如果想延时执行代码、方法又是什么？

线程创建有三种方法：使用NSThread创建、使用GCD的dispatch、使用子类化的NSOperation,然后将其加入NSOperationQueue;在主线程执行代码，方法是performSelectorOnMainThread，如果想延时执行代码可以用performSelector:onThread:withObject:waitUntilDone:

### 12、写出一段代码，计算正整数123的各位数字之和。

### 13、写出冒泡排序算法，你还知道哪些排序算法？

\#include <stdio.h>

int main(void)

{

int num[10] = {};

int i = 0;

int j = 0;

int temp;

for(i = 0; i < 10; i++)

{

scanf("%d", &num[i]);

}

for(i = 0; i < 9; i++)

{

for(j = 0; j < 9 - i; j++)

{

if(num[j] > num[j + 1])

{

temp = num[j];

num[j] = num[j + 1];

num[j + 1] = temp;

}

}

}

for(i = 0; i < 10; i++)

{

printf("%d\n", num[i]);

}

return 0;

}

# 北京东方国际科技有限公司

iOS笔试题

…

1、简答题

a、请写出ViewController的viewWillAppear，viewDidLoad，loadView，ViewDidUnLoad调用的先后顺序。在自定义loadView时需要注意些什么？

b、分别描述委托（protocol）、target、本地通知（notification）的区别和使用方式。

c、描述alloc、retain、copy、assign、autorelease、release、dealloc各自的作用。自动释放池的生命周期，举例说明需要手动创建自动释放池的场景。

d、如何渐变填充多边形？（如：亮度从中心到五角渐暗的五角星）

a、（1）永远不要主动调用 loadView 方法

（2）永远不要在覆盖 loadView 方法时使用 [super loadView]

（3）在 loadView 中实例化 view，在 viewDidLoad 中自定义 view

b、用delegate的时机：

|需要一对一发送消息

|对性能有需求

|只需要调用一个方法

|需要被调用的方法可以‘影响’该对象

\-----------------------------------

|用Notification的时机：

|需要一对多广播消息

|程序间通信

\-------------------------

|用Target-Action的时机：

|发送对象是系统控件类(Button.etc.)

|（大体上）都是对应系统事件的，比如Click，Drag什么的

|可以1vN，也就是同一个button click事件可以addTarget多次|

\--------------------------------------------------------------

|delegate方法都是由协议（Protocol）声明的，剩下俩都不需要声明,因为他们用的都是@selector();

c、OC中内存管理机制应该就是引用计数的增减吧，retainCount为0时释放该内存。

retain对应的是release，内存的释放用release。

alloc对应的是dealloc，内存的销毁用dealloc。

readwrite此标记说明属性会被当成读写的，这也是默认属性。

readonly此标记说明属性只可以读，也就是不能设置，可以获取。

assign不会使引用计数加1，也就是直接赋值。

retain会使引用计数加1。

copy建立一个索引计数为1的对象，在赋值时使用传入值的一份拷贝。

nonatomic：非原子性访问，多线程并发访问会提高性能。

atomic:原子性访问。

strong:强引用

weak:弱引用

1：自动释放池的数据结构

自动释放池是以栈的形式实现，当你创建一个新的自动释放池，它将会被添加到栈顶。接受autorelease消息的对象将会被放入栈顶

2：如何持有对象

当我们使用alloc，copy,retain对象获取一个对象时，我们需要负责显示的安排对象的销毁，其他方法获取的对象将交给自动释放池进行释放（单例模式除外）

3：release和drain的区别

当我们向自动释放池pool发送release消息，将会向池中临时对象发送一条release消息，并且自身也会呗销毁。向它发送drain消息时，只会指定前者。