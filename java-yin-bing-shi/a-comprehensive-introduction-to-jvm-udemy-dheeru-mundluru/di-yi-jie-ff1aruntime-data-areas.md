# Runtime Data Areas

### Process Virtual Address Space

交换区 \(Swap space\) 与物理内存共同组成操作系统的**虚拟内存**。当物理内存尚有空闲时，新产生的数据 \(进程及其数据\) 会直接被放在内存中；当物理内存空间不足时，操作系统会根据优先级从物理内存中把不活跃的数据取出来，暂时放入交换区，为优先级高的数据腾出空间。

虚拟内存空间主要分为两部分：用户空间 \(User Space\) 与内核空间 \(Kernel Space\)，内核空间为所有进程共用，用户空间则为每个用户进程各自使用。JVM 也是普通的用户程序，因此它也存在于用户空间中。

### JVM Runtime Data Areas

\(图1）

如上图所示，JVM 的运行时内存由所有线程共用的 Method Area、Heap \(Java Heap\)，每个线程私有的 Stack、PC、Native Method Stack，以及





