# 第二十课 - 通用机 \(Universal Machines\)

### 引入

前几节课程最终想要传达的核心思想其实就是：

> Evaluator 决定 Scheme 程序的意义

我们也可以直观地观察到

> Evaluator 自身也是个程序

也就是说我们有一个能解释其它程序意义的程序。

### 如何看待 Evaluator

#### 从语言设计者的角度出发

Evaluator 定义了解释程序的方式如：

* Applicative vs. Normal order
* Dynamic vs. Lexical scoping
* Decoupling analysis from evaluation

前两个例子不再赘述，这边介绍一下第三条，将 analysis 与 evaluation 解耦：

我们前面介绍的 Scheme 解释器的示意图如下：

![](/assets/Screen Shot 2018-04-17 at 1.36.33 PM.jpg)

这种解释器被称为 straight interpreter，因为它不会对输入的表达式作任何分析，而是直接在环境中解释表达式。但通常情况下，输入的表达式有优化空间，因此就有下面这种功能更强大的解释器或者编译器 \(advanced interpreter or compiler\)：

![](/assets/Screen Shot 2018-04-17 at 1.36.40 PM.jpg)

在 expression 进入 execution 之前，多了一个静态分析模块，它可以：

* 提升执行效率
* 及早发现常见错误
* 对表达式做安检

所有这些，都是为了减少 evaluation 的成本。course notes 还对部分表达式的 analyzer 的实现举例，这里也不再赘述。

#### 从理论研究者的角度出发

引入中提到过，Evaluator 是一个能解释其它程序的程序，它的另外一个名字就是通用机 \(Universal Machine\)。

> Universal Machine is a program that takes any program as input and reconfigures itself to simulate the input program

Evaluator 不仅能够解释其它程序，它自身也是一个程序，因此 Evaluator 也可以用来解释其它语言的 Evaluator，这也是我们从十七课开始在做的事情 --- 利用 Scheme 的 Evaluator 来构建 Scheme\* 的 Evaluator。极端地说，任何语言的 Evaluator 都可以被用来解释任何其它语言的 Evaluator，换言之，你可以用 C 语言写 Java 的、Python 的、Ruby 的....任何语言的 Evaluators。

#### 参考

* [Youtube: SICP-2004-Lecture-20](https://www.youtube.com/watch?v=G8JWoBEaWWc&index=20&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&t=0s)
* [MIT6.006-SICP-2005-Lecture-notes-20](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture23webhan.pdf)



