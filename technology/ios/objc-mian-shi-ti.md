---
coverY: 0
layout:
  cover:
    visible: false
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# objc\_面试题

## 一、视图&图像相关

### 1.1 UIView & CALayer的区别

* UIView 为 CALayer 提供内容，以及负责处理触摸等事件，参与响应链；
* CALayer 负责显示内容 contents
* 单⼀职责原则

### 1.2 layoutSubviews & setNeedsLayout &

layoutSubviews触发时机：

1. init初始化不会触发layoutSubviews。
2. addSubview会触发layoutSubviews。
3. 改变⼀个UIView的Frame会触发layoutSubviews，当然前提是frame的值设置前后发⽣了变化。
4. 滚动⼀个UIScrollView引发UIView的重新布局会触发layoutSubviews。
5. 旋转Screen会触发⽗UIView上的layoutSubviews事件。
6. 直接调⽤ setNeedsLayout 或者 layoutIfNeeded。

* setNeedsLayout 标记为需要重新布局，异步调⽤ layoutIfNeeded 刷新布局，不⽴即刷新，在 下⼀轮runloop结束前刷新，对于这⼀轮 runloop 之内的所有布局和UI上的更新只会刷新⼀ 次， layoutSubviews ⼀定会被调⽤。
* layoutIfNeeded 如果有需要刷新的标记，⽴即调⽤ layoutSubviews 进⾏布局（如果没有标 记，不会调⽤ layoutSubviews ）。

### 1.3 什么是离屏渲染

#### imageNamed & imageWithContentsOfFile区别？

imageNamed⽅法会从指定的⽂件中加载图⽚数据，并将其缓存起来，然后再把结果返回，下次再使⽤该名称图⽚的时候就省去了从硬盘中加载图⽚的过程。对于相同名称的图⽚，系统只会把它Cache到内存⼀次，如果相应的图⽚数据不存在，返回nil。

* 关于加载：imageNamed ⽅法可以加载 Assets.xcassets 和 bundle 中的图⽚。
* 关于缓存：加载到内存当中会⼀直存在内存当中，（图⽚）不会随着对象的销毁⽽销毁；加载进去图⽚ 后，占⽤的内存归系统管理，我们是⽆法管理的；相同的图⽚是不会重复加载的，加载到内存中占据的 内存较⼤

imageWithContentsOfFile ⽅法只是简单的加载图⽚，并不会将图⽚缓存起来，图像会被系统以数 据⽅式加载到程序。

* 关于加载： imageWithContentsOfFile 只能加载 mainBundle 中图⽚
* 关于缓存：加载到内存中占据的内存较⼩，相同的图⽚会被重复加载到内存当中，加载的图⽚会随着对 象的销毁⽽销毁；

两者应⽤场景：

* 如果图⽚较⼩，并且使⽤频繁的图⽚使⽤ imageName：⽅法来加载
* 如果图⽚较⼤，并且使⽤较少，使⽤imageWithContentOfFile:来加载。
* 当你不需要重⽤该图像，或者你需要将图像以数据⽅式存储到数据库，⼜或者你要通过⽹络下载⼀ 个很⼤的图像时，使⽤ imageWithContentsOfFile ；
* 如果在程序中经常需要重⽤的图⽚，⽐如⽤于UITableView的图⽚，那么最好是选择imageNamed ⽅法。这种⽅法可以节省出每次都从磁盘加载图⽚的时间；

### 1.4 图⽚渲染怎么优化

1 图⽚尺⼨明显⼤于 UIImageView 显示尺⼨的场景

## 二、Runloop

### 2.1 Runloop与绘图循环&#x20;

UIApplication在主线程初始化的Runloop是Main Runloop，负责处理app存活期间的大部分时间，一直处于不断处理事件和休眠的循环中，以确保能尽快的将用户事件传递给GPU进行渲染，使用户行为能够得到响应。

每一个view的变化的修改并不是立刻变化，相反的会在当前run loop的结束的时候统一进行重绘。设计目的是为了能够在一个runloop里面处理好所有需要变化的view，包括resize、hide、reposition等等。所有view的改变都能在同一时间生效，这样能够更高效的处理绘制，这个机制被称为绘图循环（View Drawing Cycle)。

### 2.2 Runloop与AutorealsePool



## 三、多线程

### 3.1 iOS开发中有多少类型的线程？

* NSThread，每个 NSThread对象对应⼀个线程，量级较轻，通常我们会起⼀个 runloop 保活，然 后通过添加⾃定义source0源或者 perform onThread 来进⾏调⽤，优点轻量级，使⽤简单，缺 点：需要⾃⼰管理线程的⽣命周期，保活，另外还会线程同步，加锁、睡眠和唤醒
* GCD：Grand Central Dispatch（派发） 是基于C语⾔的框架，可以充分利⽤多核，是苹果推荐使 ⽤的多线程技术
  * 优点：GCD更接近底层，⽽NSOperationQueue则更⾼级抽象，所以GCD在追求性能的底层操作来说，是速度最快的
  * 缺点：操作之间的事务性，顺序⾏，依赖关系。GCD需要⾃⼰写更多的代码来实现
* NSOperation
  * 优点： 使⽤者的关注点都放在了 operation 上，⽽不需要线程管理。
    * ⽀持在操作对象之间依赖关系，⽅便控制执⾏顺序。
    * ⽀持可选的完成块，它在操作的主要任务完成后执⾏。
    * ⽀持使⽤KVO通知监视操作执⾏状态的变化。
    * ⽀持设定操作的优先级，从⽽影响它们的相对执⾏顺序。
    * ⽀持取消操作，允许您在操作执⾏时暂停操作。
  * 缺点：⾼级抽象，性能⽅⾯相较 GCD 来说不⾜⼀些;

### 3.2 GCD主线程 & 主队列的关系？

队列其实就是⼀个数据结构体，主队列由于是串⾏队列，所以⼊队列中的 task 会逐⼀派发到主线程中 执⾏；但是其他队列也可能会派发到主线程执⾏

### 3.3 dispatch\_once实现原理？

1、dispatch\_atomic\_cmpxchg锁，原⼦操作机制，原理就是如果p==o，那么将p设置为n，然后返回true;否则，不做任何处理返回false 2、在多线程环境中，如果某⼀个线程A⾸次进⼊ dispatch\_once\_f ，val==0，这个时候直接将其 原⼦操作设为1，然后执⾏传⼊ dispatch\_once\_f 的block，然后调⽤ dispatch\_atomic\_barrier ，最后将val的值修改为~~0。 3、dispatch\_atomic\_barrier 是⼀种内存屏障，所谓内存屏障，从处理器⻆度来说，是⽤来串 ⾏化读写操作的，从软件⻆度来讲，就是⽤来解决顺序⼀致性问题的。编译器不是要打乱代码执⾏ 顺序吗，处理器不是要乱序执⾏吗，你插⼊⼀个内存屏障，就相当于告诉编译器，屏障前后的指令 顺序不能颠倒，告诉处理器，只有等屏障前的指令执⾏完了，屏障后的指令才能开始执⾏。所以这 ⾥ dispatch\_atomic\_barrier 能保证只有在block执⾏完毕后才能修改_val的值。 4、4、在⾸个线程A执⾏block的过程中，如果其它的线程也进⼊ dispatch\_once\_f ，那么这个时候if 的原⼦判断⼀定是返回false，于是⾛到了else分⽀，于是执⾏了do\~while循环，其中调⽤了 \_dispatch\_hardware\_pause ，这有助于提⾼性能和节省CPU耗电，pause就像nop，⼲的事情 就是延迟空等的事情。直到⾸个线程已经将block执⾏完毕且将_val修改为~~0，调⽤ dispatch\_atomic\_barrier 后退出。这么看来其它的线程是⽆法执⾏block的，这就保证了在 dispatch\_once\_f 的block的执⾏的唯⼀性，⽣成的单例也是唯⼀的。

写单例时要注意： 1、初始化要尽量简单，不要太复杂； 2、尽量能保持⾃给⾃⾜，减少对别的模块或者类的依赖； 3、单例尽量考虑使⽤场景，不要随意实现单例，否则这些单例⼀旦初始化就会⼀直占着资源不能释放，造成⼤量的资源浪费。

### 3.4 什么情况下会死锁？

线程死锁是指由于两个或者多个线程互相持有对⽅所需要的资源，导致这些线程处于等待状态，⽆法前 往执⾏。当线程互相持有对⽅所需要的资源时，会互相等待对⽅释放资源，如果线程都不主动释放所占 有的资源，将产⽣死锁。

### 3.5、为什么只在主线程刷新UI?

* UI库：UIKit并不是⼀个 线程安全 的类，但UIKit只能在主线程进行操作。
* 事件处理：因为整个程序的起点UIApplication是在主线程进⾏初始化，所有的⽤户事件都是在主线程上进⾏传递（如点击、拖动），所以view 只能在主线程上才能对事件进⾏响应。
* 渲染方面：图像的渲染需要以60帧的刷新率在屏幕上同时更新，在⾮主线程异步化的情况下⽆法确定这个处理过程能够实现同步更新。

### 3.6 如何使线程保活？&#x20;

runloop 线程保活前提就是有事情要处理，这⾥指 timer，source0，source1 事件。

* 方法一：处理timer事件

```objc
NSTimer *timer = [NSTimer timerWithTimeInterval:1 repeats:YES block:^(NSTimer* _Nonnull timer) {
 	NSLog(@"timer 定时任务");
}];

NSRunLoop *runloop = [NSRunLoop currentRunLoop];
[runloop addTimer:timer forMode:NSDefaultRunLoopMode];
[runloop run];
```

* 方法二：处理source0事件

```objc
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
[runLoop run]
```

## 四、KVO\&KVC

### 4.1 那些情况下KVO会崩溃，怎么防护崩溃？

* `dealloc`没有移除KVO观察者：创建一个中间对象，将其作为某个属性的观察者，然后`dealloc`的时候移除观察者，而调用者持有中间对象的，调用者释放了，中间对象也释放了，`dealloc`也就移除了观察者了。
* 多次重复移除同一属性，移除未注册的观察者
* 被观察者提前被释放，被观察者在 dealloc 时仍然注册着 KVO，导致崩溃。例如：被观察者是局部变量的情况（iOS 10 及之前会崩溃） ⽐如 weak
* 添加了观察者，但未实现`observeValueForKeyPath:ofObject:change:context:`⽅法，导致崩溃
* 添加或者移除时 keypath == nil ，导致崩溃

## 五、Block

### 5.1 block是类吗，有哪些类型？

Block是封装了函数及上下文的类。block有三种：`NSConcreteGlobalBlock`、`NSConcreteStackBlock`、`NSConcreteMallocBlock`，根据Block对象创建时所处数据区不同⽽进⾏区别。

* 全局Block，未引⽤任何栈上变量时就是全局Block;
* 栈上Block，引⽤了栈上变量，⽣命周期由系统控制的，⼀旦所属作⽤域结束，就被系统销毁
* 堆上 Block，使⽤ copy 或者 strong（ARC）下就从栈Block 拷⻉到堆上。

### 5.2 ⼀个 int 变量被 `__block`修饰与否的区别？block的变量截获?

值 copy 和指针 copy， \_\_block 修饰的话允许在 block 内部修改变量，因为传⼊的是 int变量的 指针

外部变量有四种类型：⾃动变量、静态变量、静态全局变量、全局变量。

### 5.3 解决循环引⽤时为什么要⽤ \_\_strong、\_\_weak 修饰

\_\_weak 就是为了避免 retainCycle，⽽block 内部 \_\_strong 则是在作⽤域 retain 持有当前对象 做⼀些操作，结束后会释放掉它。

### 5.4 block 发⽣ copy 时机?

block 从栈上拷⻉到堆上⼏种情况：

* 调⽤Block的copy⽅法
* 将Block作为函数返回值时
* 将Block赋值给\_\_strong修饰的变量或Block类型成员变量时
* 向Cocoa框架含有usingBlock的⽅法或者GCD的API传递Block参数时
