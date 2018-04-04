# Class 和 Interface 的生命周期

Class 和 Interface 统称为 Type，Java 程序的运行就是不同的 Types 之间互相交流的过程。然而，这些 Types 首先要被载入到 JVM 运行时中，才能开始参与和其它 Type 的交流。读完本节，如果能够大致回答以下问题：

* Types 是如何被载入

* 不同的 Type 按什么顺序被载入等

则本节的目的也就达到了。

### 总览

假设有一个 Hello.class 被 JVM 第一次载入，它的整个载入过程如下图所示：![](/assets/Screen Shot 2018-04-04 at 10.40.59 PM.jpg)

按照箭头顺序，每个环节概述如下

1. Class Loader 在 classpath 上寻找 Hello.class，找到后就把相应的 bytecode 载入到内存中，并生成一个 Class \(java.lang.Class\) 对象，这个 Class 对象保存着 Hello 的元信息，如 class 名称、super class 的名称、class 中的各种方法等等。这个 Class 对象只会在 Hello 第一次被载入到 JVM 的时候在内存中生成，之后则会直接使用内存中的 Class 对象。最后，Class 对象被传递给 Linking 环节。
2. Linking 环节分为三步：首先 bytecode verifier 会检查 Class 的代码是否符合要求，有无安全隐患；接着在 Preparation 阶段，JVM 为 Hello 分配静态变量空间；最后进入 Resolution 阶段，JVM 会载入 Hello Class 引用的其它 Class。值得一提的是，Resolution 阶段不一定会在 Linking 环节发生，它也可以在第 3 步后发生。
3. Initialization 环节会初始化 Hello 的静态变量，无论是赋予初始值还是用 static initialization block 来初始化。

除此之外，Class Loader 总是先载入父类后才会载入子类，因此 Hello 的父类如果不在内存中，那么在载入 Hello 之前，Hello 的父类将先被载入。

如果载入的是一个 Interface，那么它将不会被立即初始化。其它部分与 Class 相同。

### Class Loading

Class loading 的逻辑很简洁，如果要找的 Class object 已经在 JVM heap 中，那么直接返回 heap 中的 Class object；如果没有，则去 classpath 中搜索对应的 .class 文件，若找到则读取到内存中，创建相应的 Class object，并放到 JVM 的 heap 中；若未找到则抛出 ClassNotFoundException。具体流程如下图所示：![](/assets/Screen Shot 2018-04-04 at 10.41.26 PM.jpg)

前面提到，如果是第一次载入，Class Loader 需要去相应路径搜索对应的 class bytecode。这个路径的搜索过程是按照从信任源到不信任源的顺序进行，如下图所示：![](/assets/Screen Shot 2018-04-04 at 10.42.04 PM.jpg)

Bootstrap Class Loader 负责载入 $java\_home$/jre/lib/rt.jar 中的 Class，这里是 Java Platform 的标准库，因此 JVM 会完全信任来自这里的 Class，在 Linking 阶段不再做 verification；Application Class Loader 负责载入用户本地 $classpath$ 上的 Class，因为这里的源不受信任，因此在 Linking 阶段会对来自于这里的 Class 做充分的 verification。

##### 什么时候 Class 会被载入

* 创建 Class 的实例
* 调用 Class 的 static method
* 访问 Class 的 static field （compile-time constants 除外）
* Class 的子类被载入
* 从命令行运行 Class
* Reflection

##### 什么时候 Interface 会被载入

* 调用 Interface 的 static method
* 访问 Interface 的 static field \(compile-time constants 除外\)
* Interface 的 subInterface 被载入
* 从命令行运行 Interface
* Reflection

### Class Object

Java 使用 Class object 创建对应的实例。事实上，Java 中的 Class, Interface, Primitives, void, Arrays 都有对应的 Class object，维度、类型相同的数组，其对应的 Class object 也相同。Class object 包含了 Class 本身的许多元信息，如：

```java
String getName();
Class getSuperClass();
boolean isInterface();
Class[] getInterfaces();
ClassLoader getClassLoader();
```

这些都是 Class object 的 instance methods。

### Linking

Linking 主要分为 Verification、Preparation 以及 Resolution 三步。

#### Verification

Verification 主要出于安全考虑，对来自于非信任源的 class bytecode 进行多重检查，防止这些 class 执行不安全的语句。检查的内容如：

* final class 不能被继承
* final methods 不能被重载

#### Preparation

Preparation 阶段主要负责为载入的 class 分配 static fields 的空间，并且初始化它们的值，如果内存空间不足，Preparation 阶段将抛出 OutOfMemoryError。如果载入 class 是为了创建实例，本阶段也会为 instance fields 分配空间。需要注意的是，class 的载入一定会发生在其 superclass 载入之后。

#### Resolution

Resolution 阶段主要 resolve 的是 class 所依赖的别的 class。本阶段会将别的 class 的 symbolic reference 存在 .class 文件的 constant pool 中，在实际运行阶段，这些 symbolic reference 会被替换成真正的内存地址，这个过程被称为 dynamic linking，即在运行时才真正确定所依赖的 classes。Dynamic linking 可以很容易做到动态更新，而无需重新编译源码；但如果所更新的部门与原系统不兼容，则可能导致系统崩溃。Resolution 可以发生在 linking 阶段，也可以发生在 initialization 阶段之后，前者被称为 eager loading，无论依赖的 classes 是否最终被用到，都会将依赖载入；后者被称为 lazy loading，直到真正需要的时候才去载入依赖。同时，Resolution 还会做一些安全检查，比如所依赖的 class 是否有 class 使用的变量、方法，以及 class 是否有使用所依赖 class 的权限等等。







