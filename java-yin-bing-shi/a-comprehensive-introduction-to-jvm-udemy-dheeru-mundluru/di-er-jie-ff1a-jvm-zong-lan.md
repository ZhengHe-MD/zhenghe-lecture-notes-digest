# JVM 总览

### JVM 的 V

我们知道，一段 C 语言要执行，会先经过编译器编译成为机器码然后执行。这些机器码由计算机架构的指令集构成，不同架构如 x86、ARM 各自拥有不同的指令集，它们是 Computing Machine。Java Virtual Machine 也是 Computing Machine，但它的名字里多了一个 Virtual，这显然意味着 JVM 肯定在计算机架构级别上增加了一层抽象。事实上：

* JVM 有自己的指令集，而 Java 字节码正由这些指令集构成
* JVM 有自己的内存区域，在运行过程中自行完成内存操作

### JVM 的主要职责

* 加载并解释 Java 字节码
* 保证代码执行的安全性 （Java 要支持加载和解释网络上传输的代码）
* 自动内存管理

### JVM 的规范 \(Specification\) 与实现 \(Implementation\)

JVM 的规范定义了虚拟机的各种概念和行为，如指令集、内存管理等。每一个版本的 JVM 规范旁边也会有同一版本的 Java 规范，具体可以参见 [Java SE Specification](https://docs.oracle.com/javase/specs/)。相对于规范， JVM 的实现则是按照规范构建的 JVM 实体，如 Oracle's HotSpot JVM 以及 IBM's JVM。

当我们在命令行执行

```
$ java HelloWorld
```

1. 启动一个 JVM 运行时
2. JVM 运行时加载 HelloWorld.class
3. 执行 HelloWorld 的 main 函数

### JVM 的性能保证

* 对 Java 字节码的解释速度极快
* Just-in-time \(JIT\) 编译，也称为动态编译 \(dynamic compilation\)
  * 找到频繁执行的字节码，即所谓热点 \(hot spots\)
  * JIT 将热点直接转化成机器码并缓存起来，未来再次执行则无需再次解释热点字节码。

需要注意的是，JIT 并不在 JVM 的规范中规定，但基本上现有的实现都使用这种方式优化性能。

### JVM 的架构![](/assets/Screen Shot 2018-02-05 at 10.08.09 PM.jpg)

**Class Loader**: 负责将程序所需的 .class 字节码载入 JVM 实例

**Bytecode Verifier**: 负责验证载入的 class 文件是否符合规范

**Execution Engine**: 由解释器和 JIT 编译器组成，负责解释和动态编译 Bytecode 程序

**Runtime Data Areas**: 负责存储运行时的程序和数据

**Garbage Collector**: 负责自动回收不再使用的内存

**Security Manager**: 负责在 sandbox 中运行可能有安全问题的代码



