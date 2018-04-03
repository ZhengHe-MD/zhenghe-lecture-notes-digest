# Class 和 Interface 的生命周期

Class 和 Interface 统称为 Type。

### 总览

假设有一个 Hello.class 被 JVM 第一次载入，它的整个载入过程如下图所示：

（图1）

按照箭头顺序，每个环节概述如下

1. Class Loader 在 classpath 上寻找 Hello.class，找到后就把相应的 bytecode 载入到内存中，并生成一个 Class \(java.lang.Class\) 对象，这个 Class 对象保存着 Hello 的元信息，如 class 名称、super class 的名称、class 中的各种方法等等。这个 Class 对象只会在 Hello 第一次被载入到 JVM 的时候在内存中生成，之后则会直接使用内存中的 Class 对象。最后，Class 对象被传递给 Linking 环节。
2. Linking 环节分为三步：首先 bytecode verifier 会检查 Class 的代码是否符合要求，有无安全隐患；接着在 Preparation 阶段，JVM 为 Hello 分配静态变量空间；最后进入 Resolution 阶段，JVM 会载入 Hello Class 引用的其它 Class。值得一提的是，Resolution 阶段不一定会在 Linking 环节发生，它也可以在第 3 步后发生。
3. Initialization 环节会初始化 Hello 的静态变量，无论是赋予初始值还是用 static initialization block 来初始化。

除此之外，Class Loader 总是先载入父类后才会载入子类，因此 Hello 的父类如果不在内存中，那么在载入 Hello 之前，Hello 的父类将先被载入。

如果载入的是一个 Interface，那么它将不会被立即初始化。其它部分与 Class 相同。

#### Class Loading

class loading 的逻辑很简洁，如果要找的 Class object 已经在 JVM heap 中，那么直接返回 heap 中的 Class object；如果没有，则去 classpath 中搜索对应的 .class 文件，若找到则读取到内存中，创建相应的 Class object，并放到 JVM 的 heap 中；若未找到则抛出 ClassNotFoundException。具体流程如下图所示：

（图2）

前面提到，如果是第一次载入，Class Loader 需要去相应路径搜索对应的 class bytecode。这个路径的搜索过程是按照从信任源到不信任源的顺序进行，如下图所示：

（图3）

Bootstrap Class Loader 负责载入 $java\_home$/jre/lib/rt.jar 中的 Class，这里是 Java Platform 的标准库，因此 JVM 会完全信任来自这里的 Class，在 Linking 阶段不再做 verification；Application Class Loader 负责载入用户本地 $classpath$ 上的 Class，因为这里的源不受信任，因此在 Linking 阶段会对来自于这里的 Class 做充分的 verification。





