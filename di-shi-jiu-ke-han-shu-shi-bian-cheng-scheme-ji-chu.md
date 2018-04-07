# 第十九课 - 函数式编程 - Scheme 基础

### 引入

本课之前，我们介绍了两种编程范式，面向过程 \(procedure-oriented\) 和面向对象 \(object-oriented\)。面向过程范式下的程序通常由几个高级 \(high-level\) 函数如 init, doTaskA, doTaskB, finish和一些负责更小过程的低级 \(low-level\) 函数构成；面向对象范式下的程序通常围绕着系统中的对象，通过对象之间的信息传递改变对象内部状态，从而完成具体编程任务。本节将介绍一种新的范式，函数式编程 \(Functional Programming\)。

函数式编程与面向过程编程有些相似，但最大的区别在于：面向过程编程不在乎每个函数的返回结果，而函数式编程则是围绕着每个函数的返回结果来编程。函数式编程通常把一个大的编程任务看作是一个函数，任务的输入数据就是函数的输入，而任务的结果就是函数的输出结果，而这个任务对应的函数又可以分解 \(decompose\) 成许多更小的函数，它们之间同样依靠输入和输出结果来沟通，达到完成最终任务的目的，举例如下：

```
f(x, y) = x^3 + y^2 + 7
g(x) = f(x, x+1) + 8
h(x, y, z) = f(x, z) * g(x+y)
```

我们的任务是就是输入 x, y, z，输出 h\(x, y, z\) 的值，而 h\(x, y, z\) 又可以被分解成 f\(x, z\) 和 g\(x+y\) 相乘的结果...

### Scheme 举例

```scheme
> (define celsius->fahrenheit (temp)
    (+ 32 (* 1.8 temp)))
> (celsius->fahrenheit 100)
212
```

为了和面向过程编程区分，我们把 celsius-&gt; fahrenheit 称为 procedure，\(celsius-&gt;fahrenheit 100\) 称为 evaluate procedure，与二者对应的是面向过程编程中的函数 \(function\) 以及函数调用 \(function call\)。

