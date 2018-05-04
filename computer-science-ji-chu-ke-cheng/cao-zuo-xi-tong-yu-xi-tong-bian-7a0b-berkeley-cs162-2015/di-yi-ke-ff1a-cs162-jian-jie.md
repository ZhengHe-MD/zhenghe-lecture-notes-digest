# CS162 简介

#### 操作系统的基石

* Scheduling
* Concurrency
* Address spaces
* Protection, Isolation, Security
* Networking, distributed systems
* Persistent storage, transactions, consistency, resilience
* Interfaces to all devices

操作系统无处不在，举例如下：

![](/assets/Screen Shot 2018-05-04 at 1.01.05 PM.jpg)

一个搜索请求从手机发出，需要首先通过 DNS 解析域名，获得服务器 ip 地址后经过网络找到搜索引擎的服务中心。在服务中心内部，请求首先来到负载均衡器，然后传递给 web 服务器，web 服务器从索引服务器、页面服务器以及广告服务器中分别获取必要的信息后组装成一个完整的页面，再经由网络传递回手机。在这整个过程中涉及到的组件，如手机、DNS 服务器、路由器、负载均衡器、索引、广告、页面服务器，每一个都有特定的操作系统在后面指挥，保证整个过程井井有条。

#### 什么是操作系统

操作系统可以理解成一层特殊的软件，它负责为其它应用程序提供硬件资源：

* 将不同的硬件抽象出方便易用的接口
* 保护公共资源的使用合理
* 安全、用户认证
* 不同个体间的通信

##### "Virtual Machine" Boundary

![](/assets/Screen Shot 2018-05-04 at 1.01.30 PM.jpg)

OS 通过虚拟化为应用程序提供硬件资源，比如图中的 Threads、Processes、Files、Sockets 等等，应用软件并不知道它实际上使用了哪些硬件，执行了哪些硬件操作，OS 负责将虚拟资源与实际硬件资源连接起来。

##### 程序 \(Program\) 与进程 \(Process\)

![](/assets/Screen Shot 2018-05-04 at 1.01.52 PM.jpg)

一段完成某个任务的代码被称为程序，代码本身描述了完成这个任务的步骤，此时它并没有被执行。当程序被载入内存中，并在处理器中开始执行时，这个运行的程序就被称为进程。

##### 上下文切换 \(Context Switch\)

![](/assets/Screen Shot 2018-05-04 at 1.02.11 PM.jpg)

当进程 A 运行到一半，需要运行进程 B 时，处理器中进程 A 运行时的上下文将被存下，之后在将进程 B 的上下文载入处理器中，开始运行进程 B，这个过程叫做上下文切换。进程 A、B 可能是用户进程，也可能是系统进程。

##### 调度和保护 \(Scheduling, Protection\)

![](/assets/Screen Shot 2018-05-04 at 1.02.33 PM.jpg)

当进程 A、B、C 同时存在内存中时，OS 需要决定它们谁能获取处理器资源，这个过程称为调度。由于多个进程都在内存中，它们之间不应当互相读写数据，这是 OS 需要为进程提供的保护之一。Protection Boundary 将硬件、OS 与用户进程分隔开，Boundary 以下的部分需要特殊保护，这里暂时不深入讨论。

##### 输入与输出 \(I/O\)

![](/assets/Screen Shot 2018-05-04 at 1.02.52 PM.jpg)

操作系统提供输入设备与输入设备的抽象，让输入与输出的使用接口更加友好而统一。比如对于应用程序来说，闪存中的文件与硬盘中的文件并无区别。

##### 载入 \(Loading\)

![](/assets/Screen Shot 2018-05-04 at 1.03.09 PM.jpg)

通常程序存储在外存设备 \(storage\) 中，在运行前它需要被读取到内存中，这个过程叫作载入。

#### 操作系统的挑战

* 处理器多核化
* 存储能力激增
* 网络传输能力增长
* 互联网规模膨胀
* 物联网

#### 课程提纲

* OS Concepts: How to Navigate as a Systems Programmer!
  * Process, I/O, Networks and VM
* Concurrency:
  * Threads, scheduling, locks, deadlock, scalability, fairness
* ·Address Space:
  * Virtual memory, address translation, protection, sharing
* File Systems
  * i/o devices, file objects, storage, naming, caching, performance, paging, transactions, databases
* Distributed Systems \(8\)
  * Protocols, N-Tiers, RPC, NFS, DHTs, Consistency, Scalability, multicast
* Reliability & Security
  * Fault tolerance, protection, security
* Cloud Infrastructure

#### 参考

* [Youtube: CS162-1](https://www.youtube.com/watch?v=qcyXohw1H00&list=PL--jIyXjDXf6Q4XA6q8RYnyChYzJ0K0F2&t=0s&index=1)
* [CS162-Spring-2015](https://inst.eecs.berkeley.edu/~cs162/sp15/)



