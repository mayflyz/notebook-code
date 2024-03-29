# 操作系统—进程管理

## 一、用户态和内核态 <a href="#tid-zrmp2c" id="tid-zrmp2c"></a>

根据进程访问资源的特点，将进程在系统上的运行分为两个级别：

* **用户态(User Mode)**：用户态运行线程权限较低，可直接读取用户用户程序的数据。当需要某些特殊权限的操作时，进入内核态。
* **内核态(Kernel Mode)**：内核态的进程几乎可以访问计算机的任何资源，拥有非常高的权限。

同时具有用户态和内核态主要是为了保证计算机系统的安全性、稳定性和性能。

#### 用户态切换到内核态方式：

* **系统调用(Trap)**：用户态进程 **主动** 要求切换到内核态的一种方式，核心是使用操作系统的一个中断来实现
* **中断(Interrupt)**：当外围设备完成用户请求的操作后，会向 CPU 发出相应的中断信号，这时 CPU 会暂停执行下一条即将要执行的指令转而去执行与中断信号对应的处理程序
* **异常(Exception)**：当 CPU 在执行运行在用户态下的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中

#### 系统调用

在用户态进程中，凡是与系统态级别的资源有关的操作，都必须通过系统调用方式向操作系统提出服务请求，并由操作系统代为完成。

系统调用的过程如下：

1. 用户态程序发起系统调用，会产生中断，即Trap
2. 中断后，当前CPU执行的程序会中断，跳转到中断处理程序。内核程序开始执行，即开始处理系统调用
3. 内核态处理完成后，主动触发Trap，再次发生中断，切换回用户态工作

## 二、进程和线程

进程是资源分配的基本单位，线程是独立调度的基本单位。

> 进程控制块 (Process Control Block, PCB) 描述进程的基本信息和运行状态，所谓的创建进程和撤销进程，都是指对 PCB 的操作。
>
> 一个进程中可以有多个线程，它们共享进程资源

进程和线程的区别：

* 拥有资源：进程是资源分配的基本单位；线程不拥有资源，可以访问隶属进程的资源
* 调度：线程是独立调度的基本单位，同一进程中，线程的切换不会引起进程切换。从一个进程中的线程切换到另一进程中的线程时，会引起进程切换
* 系统开销：创建或撤销进程，系统都要为之分配或回收资源，所付出的开销远大于创建或撤销线程时的开销。
* 通信方面：线程间可以通过直接读写同一进程的数据进行通信，进程间通信需要借助IPC

### 1、进程状态及切换

进程大致分为5种状态：

* **创建状态(new)**：进程正在被创建，尚未到就绪状态
* **就绪状态(ready)**：进程已处于准备运行状态，即进程获得除CPU外所需资源，一旦得到CPU资源(分配的时间片)即可运行
* **运行状态(running)**：进程正在处理器上运行
* **阻塞状态(waiting)**：进程正在等待某一事件而暂停运行如等待某资源为可用或等待 IO 操作完成。
* **结束状态(terminated)：**进程正在从系统中消失。可能是进程正常结束或其他原因中断退出运行**。**

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>进程状态转换图</p></figcaption></figure>

> 只有就绪态和运行态可以相互转换，其他都是单向转换。
>
> 阻塞状态是缺少需要的资源从而由运行状态转换而来，但该资源不包括CPU时间

### 2、进程调度算法

不同环境的调度算法目标不同，因此需要针对不同环境来讨论调度算法。

#### 2.1 批处理系统

批处理系统没有太多用户操作，在该系统中调度算法目标是保证吞吐量和周转时间

* **先到先服务调度算法(FCFS，First Come, First Served)：**非抢占式的调度算法，按照请求的顺序进行调度。有利于长作业，不利于短作业
* **短作业优先的调度算法(SJF，Shortest Job First)：**非抢占式的调度算法，按估计运行时间最短顺序进行调度。长作业有可能会饿死，处于一直等待短作业执行完毕的状态。
* **最短剩余时间优先 shortest remaining time next（SRTN）：**抢占式的调度算法，按剩余运行时间的顺序进行调度。

#### 2.2 交互式系统

交互式系统有大量的用户交互操作，在该系统中调度算法目标是快速地进行响应。

* **时间片轮转**：将所有就绪进程按 FCFS 的原则排成一个队列，每次调度时，把 CPU 时间分配给队首进程，该进程可以执行一个时间片。当时间片用完时，由计时器发出时钟中断，调度程序便停止该进程的执行，并将它送往就绪队列的末尾，同时继续把 CPU 时间分配给队首的进程。
* **优先级调度**：为每个进程分配一个优先级，按优先级进行调度。为了防止低优先级的进程永远等不到调度，可以随着时间的推移增加等待进程的优先级。
* **多级反馈队列**：需要连续执行多个时间片的进程考虑，它设置了多个队列，每个队列时间片大小都不同

#### 2.3 实时系统

实时系统要求一个请求在一个确定时间内得到响应。

分为硬实时和软实时，前者必须满足绝对的截止时间，后者可以容忍一定的超时。

### 3、进程管理

进程通信是手段，进程同步时目的。为了能够达到进程同步的目的，需要让进程进行通信，传输进程同步所需要的信息：

* **进程同步：**控制多个进程按一定顺序执行
* **进程通信**：进程间传输信息

#### 3.1 进程同步

进程间同步方式：

* **临界区：**对临界资源进行访问的那段代码称为临界区。为了互斥访问临界资源，每个进程在进入临界区之前，需要先进行检查
*   #### &#x20;同步与互斥

    > 同步：多个进程因为合作产生的直接制约关系，使得进程有一定的先后执行关系
    >
    > 互斥：多个进程在同一时刻只有一个进程能进入临界区
* **信号量：**一个整型变量，可以对其执行 down 和 up 操作，即P 和 V 操作.
* **管程：**管程把控制的代码独立出来

#### 3.2 进程通信

* **管道：**调用 pipe 函数创建。它只支持半双工通信(单向交替传输)，且只能在父子进程或兄弟进程中使用
* **FIFO：**命名管道。常用于客户-服务器应用程序，FIFO用作汇聚点，在客户进程和服务器进程间传递数据
* **消息队列：**
* **信号量：**用于为多个进程提供对共享数据对象的访问。
* **共享存储：**多个进程共享一个给定的存储区。使用信号量来同步对共享存储的访问。
* **套接字：**可用于不同机器间的进程通信

## 三、死锁

死锁（Deadlock）：多个进程因循环等待资源而造成无法执行的现象。

#### 产生死锁的四个必要条件： <a href="#tid-6mxmes" id="tid-6mxmes"></a>

* **互斥使用：**资源必须处于非共享模式，即一次只有一个进程可以使用
* **占有并等待：**一个进程至少应该占有一个资源，并等待另一资源，而该资源被其他进程所占有
* **不可抢占：**资源不能被抢占。只能在持有资源的进程完成任务后，该资源才会被释放。
* **循环等待**：发生死锁时，必然存在一个进程——资源的环形链

**解决死锁的方法**，一般情况下，有预防、避免、检测和解除四种：

* **预防**是采用某种策略，**限制并发进程对资源的请求**，从而使得死锁的必要条件在系统执行的任何时间上都不满足。
* **避免**则是系统在分配资源时，根据资源的使用情况**提前做出预测**，从而**避免死锁的发生**
* **检测**是指系统设有**专门的机构**，当死锁发生时，该机构能够检测死锁的发生，并精确地确定与死锁有关的进程和资源
* **解除** 是与检测相配套的一种措施，用于**将进程从死锁状态下解脱出来**

## 四、锁

锁要解决的是线程之间争取资源的问题，主要有以下几个角度：

* 资源是否是独占（独占锁 - 共享锁）
* 抢占不到资源怎么办（互斥锁 - 自旋锁）
* 自己能不能重复抢（重入锁 - 不可重入锁）
* 竞争读的情况比较多，读可不可以不加锁（读写锁）

### 4.1 独占锁 - 共享锁

当一个共享资源只有一份的时候，通常我们使用独占锁，即常见的Mutex。当共享资源有多份时，可以使用前面提到的 Semaphere。

### 4.2 互斥锁 - 自旋锁

**互斥锁：**一个线程已经锁定了一个互斥锁，第二个线程又试图去获得这个互斥锁，则第二个线程将被挂起

**自旋锁：**当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

### 4.3 重入锁 - 不可重入锁

递归锁，指的是在同一线程内，外层函数获得锁之后，内层递归函数仍然可以获取到该锁

使用可重入锁时，在同一线程中多次获取锁，不会导致死锁。使用不可重入锁，则会导致死锁发生。

### 4.4 读写锁

读写锁允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞。简单可以总结为，读读不互斥，读写互斥，写写互斥。

## 五、常见面试题

#### 1、并行和并发有什么区别?

并发就是在一段时间内，多个任务都会被处理；但在某一时刻，只有一个任务在执行。单核处理器做到的并发，其实是时间片的轮转。

并行就是在同一时刻，有多个任务在执行。多核处理器。

#### 2、什么是进程上下文切换？

上下文切换是一种将 CPU 资源从一个进程分配给另一个进程的机制。在切换的过程中，操作系统需要先存储当前进程的状态 (包括内存空间的指针，当前执行完的指令等等)，再读入下一个进程的状态，然后执行此进程。

#### 3、什么是僵尸进程？

僵尸进程是已完成且处于终止状态，但在进程表中却仍然存在的进程。

僵尸进程一般发生有父子关系的进程中，一个子进程的进程描述符在子进程退出时不会释放，只有当父进程通过 wait() 或 waitpid() 获取了子进程信息后才会释放。如果子进程退出，而父进程并没有调用 wait() 或 waitpid()，那么子进程的进程描述符仍然保存在系统中。

#### 4、什么是孤儿进程？

一个父进程退出，而它的一个或多个子进程还在运行，那么这些子进程将成为孤儿进程。

#### 5、进程和线程的联系和区别？ <a href="#jin-cheng-he-xian-cheng-de-lian-xi-he-qu-bie" id="jin-cheng-he-xian-cheng-de-lian-xi-he-qu-bie"></a>

**联系：**

> 线程是进程当中的⼀条执⾏流程。
>
> 同⼀个进程内多个线程之间可以共享代码段、数据段、打开的⽂件等资源，但每个线程各⾃都有⼀套独⽴的寄存器和栈，这样可以确保线程的控制流是相对独⽴的。

**区别**：

> 调度：**进程是资源（包括内存、打开的⽂件等）分配的单位，线程是 CPU 调度的单位**
>
> 资源：进程拥有⼀个完整的资源平台，⽽线程只独享必不可少的资源，如寄存器和栈；
>
> 拥有资源：线程同样具有就绪、阻塞、执⾏三种基本状态，同样具有状态之间的转换关系；
>
> 系统开销：线程能减少并发执⾏的时间和空间开销——创建或撤销进程时，系统都要为之分配或回收系统资源

#### &#x20;
