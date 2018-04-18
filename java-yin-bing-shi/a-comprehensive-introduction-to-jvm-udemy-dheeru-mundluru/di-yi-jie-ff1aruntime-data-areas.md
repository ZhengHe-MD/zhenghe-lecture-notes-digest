# Runtime Data Areas

### Process Virtual Address Space

交换区 \(Swap space\) 与物理内存共同组成操作系统的**虚拟内存**。当物理内存尚有空闲时，新产生的数据 \(进程及其数据\) 会直接被放在内存中；当物理内存空间不足时，操作系统会根据优先级从物理内存中把不活跃的数据取出来，暂时放入交换区，为优先级高的数据腾出空间。

虚拟内存空间主要分为两部分：用户空间 \(User Space\) 与内核空间 \(Kernel Space\)，内核空间为内核程序所用，后者负责与硬件交流、进程调度以及分配内存等工作，为用户空间中的各进程提供支持；用户空间则为每个用户进程共同使用，当用户进程需要使用系统资源时，可以通过系统调用 \(system call\) 来启动内核程序，后者将使用请求发送给对应的资源，这些请求包括文件读写、网络数据读写、创建新进程等等。

JVM 也是普通的用户程序，因此它也存在于用户空间中，如果 JVM 的 Java Heap 被放到交换区，则 JVM 的 Garbage Collector 的性能将受严重影响，甚至 Java 程序可能不再响应，因此系统为 JVM 分配的内存空间需要尽量满足 JVM 的运行内存空间。

### JVM Runtime Data Areas

![](/assets/Screen Shot 2018-04-18 at 9.38.25 PM.jpg)

如上图所示，JVM 的运行时内存由所有线程共用的 Method Area、Heap \(Java Heap\)，每个线程私有的 Stack、PC、Native Method Stack，以及 JIT Code、Direct Buffer 用到的额外的 Native Memory 共同组成。注意，Native Memory = User space - Java Heap。

* Java Heap 用来存储 Java objects，包括 Class object 、arrays 等。Java Heap 的大小可以用 java 命令行的选项来控制
* Method Area 用来存储 Class data，以及 Class 中各种 methods 的 bytecode
* JVM Stack 用于记录 method 调用的状态，局部变量信息，Native Method Stack 与 JVM Stack 类似，不过是用来记录 Native method 的相关信息。PC 用于记录下一个命令的内存地址，通常指向 Method Area 的 bytecode。JVM stack 、PC 和 Native Method Stack 都是每个线程独有的
* JIT Code 是 JIT Compiler 编译的经常被执行的代码，这些代码同样需要存储在 Memory 中
* Direct Buffer 是为了提高 IO 效率在 JVM 与 OS 的 IO 之间建立的一条捷径，在高性能缓存中发挥重要作用，通过 Direct Buffer，JVM 直接在 Native Memory 中请求 IO 空间，而不在 Java Heap 中分配相关空间。

#### 参考

* [Thanks for the memory, Linux](https://www.ibm.com/developerworks/library/j-nativememory-linux/)



