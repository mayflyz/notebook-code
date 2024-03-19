# objc\_基础

## 1、属性关键字

属性（property）用于封装对象中的数据。令编译器自动生成属性相关的存取方法，并且保存为各种实例变量。

属性本质是实例变量与存取方法的结合。`@property = ivar + getter + setter`

**属性关键字类型：**

* 原子性：`automic(默认)`、`nonatomic`
* 读写权限：`readonly`、`readwrite(默认)`
* 内存管理：`assign`、`retain`、`strong(默认)`、`copy`、`weak`、`unsafe_unretained`
* 方法名：`getter`、`setter`

> atomic原子性，编译器会自动生成互斥锁，保证属性赋值和取值安全，不包括操作和访问
>
> assign修饰基本数据类型、weak指向但不拥有该对象，不增加引用计数、unsafe\_unretained销毁时不自动清空，容易形成野指针

* **synthesize**：编译时自动生成 setter、getter方法和\_ivar实例变量，可指定实例变量名字
* **dynamic**：getter、setter在运行时产生

## 2、NSDictiony实现

NSDictionary（字典）是使用 hash表来实现key和value之间的映射和存储的。

NSDictionary使用NSMapTable实现，NSMapTable同样是一个key－value的容器。

```objectivec
typedef struct {
   NSMapTable        *table;   //table对象自身的指针
   NSInteger          i;       //计数值
   struct _NSMapNode *j;       //节点指针
} NSMapEnumerator;
```

NSMapTable是一个**哈希**＋链表的数据结构，在NSMapTable中插入或者删除一对对象时， 寻找的时间是O(1)＋O(m)，m最坏时可能为n。

NSMapTable使用**NSObject的哈希函数**：

```objectivec
-(NSUInteger)hash {
   return (NSUInteger)self>>4;
}
```

NSDictionary 提供了 key -> object 的映射。存储object的位置由key值来索引，hash值是NSObject的哈希函数，因此NSDictionary 会始终copy key 到自己私有空间。而且object对象在内部是强引用。

key 的copy行为也是 NSDictionary 如何工作的基础，这就限制了只能使用OC对象作为NSDictionary的key，并且必须支持NSCopying协议。

## 3、NSNotification实现

通知是iOS对观察者模式的实现，通知通过NSNotificatinonCenter单例来管理及通知观察者。

NSNotificatinonCenter介绍：

> * `NSNotificatinonCenter`是使用观察者模式来实现的用于跨层传递消息，用来降低耦合度。
> * `NSNotificatinonCenter`用来管理通知，将观察者注册到`NSNotificatinonCenter`的通知调度表中，然后发送通知时利用标识符`name`和`object`识别出调度表中的观察者，然后调用相应的观察者的方法，即传递消息。如果是基于`block`创建的通知就调用`NSNotification`的`block。`

**NSNotification介绍：**

> `NSNotification`是方便`NSNotificationCenter`广播到其他对象时的封装对象，简单讲即通知中心对通知调度表中的对象广播时发送`NSNotification`对象。

通知中心广播发送通知是通过`name`和`object`来确定来标识观察者，NSNotificatinonCenter调度表根据`name`和`object`来设计。定义了两个Table，一张用于保存添加观察者的时候传入了NotifcationName的情况。一张用于保存添加观察者的时候没有传入了NotifcationName的情况。

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="563"><figcaption><p>NSNotification <strong>Named Table</strong></p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3).png" alt="" width="563"><figcaption><p>NSNotification <strong>Namless Table</strong></p></figcaption></figure>

**wildcard**\
wildcard是链表的数据结构，如果在注册观察者时既没有传入NotificationName，也没有传入object，就会添加到wildcard的链表中。注册到这里的观察者能接收到 所有的系统通知。

**NSNotificationQueue**是notification Center的缓冲池，遵循FIFO的顺序转发通知给`NSNotificationCenter。`每个线程都有一个默认的NSNotificationQueue。

## 4、NSMutableArray实现



## 5、对象初始化

对象初始化调用**`alloc`**是开辟内存空间，**`init`**函数返回当前对象，**`new`**函数包装了`allo`和init方法。

#### alloc的实现流程

<mark style="color:red;">**alloc 核心作用**</mark><mark style="color:red;">就是开辟内存，初始化</mark><mark style="color:red;">`isa`</mark> <mark style="color:red;"></mark><mark style="color:red;">指针，并将对象与类isa绑定。alloc的调用流程如下：</mark>`alloc——>_objc_rootAlloc——>callAlloc——> _objc_rootAllocWithZone`_`——>_class_createInstanceFromZone`_

_`_class_createInstanceFromZone`函数内部步骤：_

1. cls->instanceSize：计算出需要的内存大小
2. calloc：向系统申请开辟内存，返回地址指针
3. objc->initInstanceIsa：关联到相应的类

<figure><img src="../.gitbook/assets/image.png" alt="" width="563"><figcaption><p>alloc底层流程</p></figcaption></figure>

objc调用函数定义依次如下：

```cpp
//1、 alloc
+ (id)alloc {
    return _objc_rootAlloc(self);
}

//2、_objc_rootAlloc
id  _objc_rootAlloc(Class cls){
    return callAlloc(cls, false/*checkNil*/, true/*allocWithZone*/);
}

//3、callAlloc
static ALWAYS_INLINE id callAlloc(Class cls, bool checkNil, bool allocWithZone=false)
{
#if __OBJC2__
    if (slowpath(checkNil && !cls)) return nil;
    if (fastpath(!cls->ISA()->hasCustomAWZ())) {
        return _objc_rootAllocWithZone(cls, nil);
    }
#endif

    // No shortcuts available.
    if (allocWithZone) {
        return ((id(*)(id, SEL, struct _NSZone *))objc_msgSend)(cls, @selector(allocWithZone:), nil);
    }
    return ((id(*)(id, SEL))objc_msgSend)(cls, @selector(alloc));
}

//4、_objc_rootAllocWithZone
NEVER_INLINE id _objc_rootAllocWithZone(Class cls, malloc_zone_t *zone __unused)
{
    // allocWithZone under __OBJC2__ ignores the zone parameter
    return _class_createInstanceFromZone(cls, 0, nil,
                                         OBJECT_CONSTRUCT_CALL_BADALLOC);
}

//5、_class_createInstanceFromZone
static ALWAYS_INLINE id _class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone,
                              int construct_flags = OBJECT_CONSTRUCT_NONE,
                              bool cxxConstruct = true,
                              size_t *outAllocatedSize = nil)
{
    ASSERT(cls->isRealized());

    // Read class's info bits all at once for performance
    bool hasCxxCtor = cxxConstruct && cls->hasCxxCtor();
    bool hasCxxDtor = cls->hasCxxDtor();
    bool fast = cls->canAllocNonpointer();
    size_t size;
    size = cls->instanceSize(extraBytes);
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    if (zone) {
        obj = (id)malloc_zone_calloc((malloc_zone_t *)zone, 1, size);
    } else {
       obj = (id)calloc(1, size);
    }
    if (slowpath(!obj)) {
        if (construct_flags & OBJECT_CONSTRUCT_CALL_BADALLOC) {
            return _objc_callBadAllocHandler(cls);
        }
        return nil;
    }

    if (!zone && fast) {
        obj->initInstanceIsa(cls, hasCxxDtor);
    } else {
        // Use raw pointer isa on the assumption that they might be
        // doing something weird with the zone or RR.
        obj->initIsa(cls);
    }

    if (fastpath(!hasCxxCtor)) {
        return obj;
    }

    construct_flags |= OBJECT_CONSTRUCT_FREE_ONFAILURE;
    return object_cxxConstructFromClass(obj, cls, construct_flags);
}
```

init函数返回当前对象，苹果这样设计允许我们队对象进行不同的配置。<mark style="color:red;">因为它就是一个工厂方法，存在是为了丰富子类的初始化。</mark>

```objectivec
- (id)init {
    return _objc_rootInit(self);
}


id _objc_rootInit(id obj)
{
    // In practice, it will be hard to rely on this function.
    // Many classes do not properly chain -init calls.
    return obj;
}
```

```objectivec
//new函数实现
+ (id)new {
    return [callAlloc(self, false/*checkNil*/) init];
}
```

## 6、类别&扩展



## 7、load\&initialize

#### 异同点：

\+load一般是用来交换方法Method Swizzle，由于它是线程安全的，而且一定会调用且只会调用一次，通常在使用UrlRouter的时候注册类的时候也在+load方法中注册。

\+initialize方法主要用来对一些不方便在编译期初始化的对象进行赋值，或者说对一些静态常量进行初始化操作。

* **相同点：**

> * load和initialize会被自动调用，不能手动调用它们
> * 子类实现了load和initialize的话，会隐式调用父类的load和initialize方法
> * load和initialize方法内部使用了锁，因此它们是线程安全的

* **不同点：**

> * 调用顺序不同，以main函数为分界，+load方法在main函数之前执行，+initialize在main函数之后执行。
> * 子类中没有实现+load方法的话，子类不会调用父类的+load方法；而子类如果没有实现+initialize方法的话，也会自动调用父类的+initialize方法。
> * \+load方法是在类被装在进来的时候就会调用，+initialize在第一次给某个类发送消息时调用（比如实例化一个对象），并且只会调用一次，是懒加载模式，如果这个类一直没有使用，就不回调用到+initialize方法。

#### +load

* \+load方法总是在main函数之前调用，在runtime阶段被调用的
* \+load方法不会覆盖。如果子类实现了+load方法，那么会先调用父类的+load方法（无需手动调用super），然后又去执行子类的+load方法。
* \+load方法只会调用一次。
* \+load方法执行顺序是：类 -> 子类 ->分类。而不同分类之间的执行顺序不一定，依据在Compile Sources中出现的顺序（先编译，则先调用，列表中在下方的为“先”）。
* \+load方法是函数指针调用，即遍历类中的方法列表，直接根据函数地址调用。如果子类没有实现+load方法，子类也不会自动调用父类的+load方法。

#### +initialize

* \+initialize方法是在类或它的子类收到第一条消息之前被调用的，这里所指的消息包括实例方法和类方法的调用。因此+initialize方法总是在main函数之后调用。
* \+initialize方法只会调用一次。
* \+initialize方法实际上是一种惰性调用，如果一个类一直没被用到，那它的+initialize方法也不会被调用，这一点有利于节约资源。
* \+initialize方法会覆盖。如果子类实现了+initialize方法，就不会执行父类的了，直接执行子类本身的。如果分类实现了+initialize方法，也不会再执行主类的。
* \+initialize方法的执行覆盖顺序是：分类 -> 子类 ->类。且只会有一个+initialize方法被执行。
* \+initialize方法是发送消息（objc\_msgSend()），如果子类没有实现+initialize方法，也会自动调用其父类的+initialize方法。

## 8、通知&代理\&KVO

* 代理是一种设计模式，一般是一对一，需要创建协议`protocol`，执行方要遵守协议，实现相应的协议方法。
* 通知是观察者模式的一种实现，用于一对多跨层传递消息。
* KVO是观察者模式的一种实现，支持一对多传递，通过isa-swizzling机制实现

通知优缺点：

> 优点：代码编写少，实现简单；一对多消息通知方式实现简单；被观察者对象和观察者对象耦合度小，context可以携带自定义消息； 缺点：编译器不会检查，需要开发者自查；调试时控制过程难跟踪；被观察者和观察者需要提前知道通知名，硬编码；通知发送消息后无反馈信息

代理优缺点：

> 优点：严格的语法，协议中有清晰的定义；控制流程可跟踪并可识别；无需第三方保持/监控通信过程；能够接受代理协议方法的返回值 缺点：代码量过多(定义协议，委托对象delegate属性，代理本身的delegate方法实现)，在释放代理对象时，循环引用导致的内存泄露。

KVO优缺点：

> 优点：提供两个对象间同步的方法；在不改变对象实现情况下，在对象状态改变时做出响应；提供了观察属性的最新值及先前值；通过keypaths来观察属性，可以观察嵌套对象 缺点：观察属性必须使用String定义；对属性对象重构导致观察失效；同一对象有多个观察值是，需要复杂的分支语句处理

## 问题

#### 1、什么是哈希表？什么是哈希冲突？

哈希表，又称散列表，通过建立键`key`与值`value`之间的映射，实现高效的元素查询。**在哈希表中进行增删查改的时间复杂度都是 O(1)。**

哈希冲突指哈希函数多个输入对应了相同输出的情况。

哈希冲突原因：哈希函数是否均匀、哈希冲突处理的方法、哈希表的负载因子`(负载因子 = 填入表中的元素个数 / 哈希表的长度)`



#### 2、通知的发送时同步的，还是异步的？

通知的发送是同步的，发送和接收通知在同一个线程。

#### 3、页面销毁时不移除通知会崩溃吗？多次添加同一通知会是什么结果？多次移除通知呢？

iOS 9后，系统使用weak指针修饰observe，当observe被释放后，再次发送消息给nil不会引起崩溃，系统会下次发送通知时移除observe为nil的观察者。

多次添加同一通知，回调会触发多次。(因为添加通知时没有做重复过滤，通过key值查到对应的链表后添加到末尾，因此会触发多次)

多次移除，无影响。现在map中查找，然后做删除。



