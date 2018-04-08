# 第十九课 - 函数式编程 - Scheme 基础

### 引入

本课之前，我们已经介绍两种编程范式，面向过程 \(procedure-oriented\) 和面向对象 \(object-oriented\)。面向过程范式下的程序通常由几个高级 \(high-level\) 函数如 init, doTaskA, doTaskB, finish和一些负责更小过程的低级 \(low-level\) 函数构成；面向对象范式下的程序通常围绕着系统中的对象，通过对象之间的信息传递改变对象内部状态，从而完成具体编程任务。本节将介绍一种新的范式，函数式编程 \(Functional Programming\)。

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

### Scheme 基础

Scheme 中，除了 primitive types，剩下的都是 list。这里 ' 实际上是 \(quote ...\) 的简写，它的作用是抑制 \(suppress\) evaluation，' 中的元素都是 list 数据。car 和 cdr 是 list 的选择器，car 返回 list 中的第一个元素，cdr 返回 list 中除第一个元素之外的 list。

```scheme
> (car '(1 2 3 4 5))
1
> (cdr '(1 2 3 4 5))
(2 3 4 5)
> (car (cdr (cdr '(1 2 3 4 5))))
3
> (caddr '(1 2 3 4 5))
3
> (cdr '(4))
()
> (cdr '())
; specification 没有定义行为，不要这么做
```

cons 是 construct 的缩写，它可以理解为 car 的逆操作，将一个元素放到 list 的首位：

```scheme
> (cons 1 '(2 3 4 5))
(1 2 3 4 5)
> (cons '(1 2 3) '(4 5))
((1 2 3) 4 5)
```

append 可以把两个 list 合成一个 list

```scheme
> (append '(1 2 3) '(4 5)
(1 2 3 4 5)
> (append '(1 2) '(3) '(4 5) '(6 7 8))
(1 2 3 4 5 6 7 8)
> (append '(2 3) (list 1) '(4 5))
```

define 可以把一个名字和一个表达式在环境中联系起来

```scheme
> (define add (x y)
    (+ x y))
ADD
> (add 10 7)
17
```

Scheme 在 runtime 中才会发现类型不匹配

```scheme
> (add "hello" "world")
error
```

Scheme 中对整个 list 的操作通常选择用递归而不是迭代来完成

```scheme
(define sum-of (numlist)
  (if (null? numlist) 0
      (+ (car numlist)
         (sum-of (cdr numlist))))
```

### 小结

Scheme 与 C 相比是一门对程序员来说更加简单的语言，它把程序员与内存直接打交道的工作完成，程序员可以更多地思考程序的逻辑本身。

#### 参考

* [Stanford-CS107-lecture-19](https://www.youtube.com/watch?v=_cV8NWQCxnE&list=PL9D558D49CA734A02&index=19&t=1s)



