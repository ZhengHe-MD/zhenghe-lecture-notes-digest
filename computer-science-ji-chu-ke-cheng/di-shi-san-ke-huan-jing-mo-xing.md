# 第十三课 - 环境模型 \(Environment Model\)

在上一节中，我们将 mutation 引入到编程工具包中，使得同一段程序运行的结果与时间或者上下文紧密相关，这使得我们的替代模型 \(substitution model\) ，也即函数式编程模型 \(functional programming model\)，不再成立。用例子回顾一下：

```scheme
>(define make-counter
    (lambda (n)
      (lambda () (set! n (+ n 1)) n)))

> (define ca (make-counter 0))
> (ca)
1
> (ca)
2

> (define cb (make-counter 0))
> (cb)
1
> (ca)
3
```

我们每次调用 make-counter 、ca 、cb 的结果都不一样，且 ca 与 cb 的调用结果相互独立。因此，我们需要另外一种模型来解释这段程序的行为，这个模型就叫做环境模型。环境模型既可以解释替代模型可以解释的代码行为，也能够解释引入 mutation 后的代码行为，它是后者的超集。

#### 几个观点的改变

在介绍环境模型之前，我们需要改进几个此前一直 take for granted 的观点

| 概念 | 旧观点 | 新观点 |
| :--- | :--- | :--- |
| 变量 \(variable\) | 数据的名字 | 放置数据的地方 |
| 程序 \(procedure\) | 将确定输入映射到确定输出的函数 | 在上下文中的一种对象 |
| 表达式 | 不需要环境也有意义 | 只在环境中存在意义 |

#### Frame: a table of bindings

我们将变量及其内部的数据合称为 binding，这些 bindings 可以组成一张表，这张表被称为 Frame。如下图所示：

![](/assets/Screen Shot 2018-03-01 at 10.45.06 PM.jpg)

从图中可以看到，在 Frame A 中，变量 x 与 15 组成一个 binding，变量 y 与 \(1 2\) 组成一个 binding。

#### Environment: a sequence of frames

环境由一系列 Frame 构成，这些 Frame 之间存在着内外包含关系 \(enclosing\)。如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.45.39 PM.jpg)

图中，环境 E1 同时包含 Frame A 和 Frame B，环境 E2 只包含 Frame B。Frame A 指向 Frame B 的箭头表示 B 是 A 的外环境，这个箭头称为外环境指针 \(enclosing environment pointer\)。既然 A 有外环境，那么 B 也可以有外环境，但这种链式关系不可能无休止地传递下去，总有一个 Frame 没有外环境指针，它是所有其它环境的最后的外环境，这个环境（或者 Frame）被称为全局环境 \(Global Environment\)。

有了环境的概念，我们就可以解释代码在环境中的执行过程。

### 环境模型

用比较机械的角度看环境模型，它实际上就是以下几个解释规则的集合：name-rule、define-rule、set!-rule、lambda-rule、application。我们不妨按顺序讨论一下几个规则：

#### name-rule

在环境 E 中的一个变量的值就是从当前 Frame 开始，顺着外环境链往上找到的第一个对应绑定的值。如图二所示，在环境 E1 中变量 x 的值就是 Frame A 中的 15，虽然 Frame B 中也含有 x 的绑定值，但 Frame A 是第一个含有 x 的绑定值的 Frame，此时我们称 Frame B 中的 x 被 Frame A 中的 x 遮盖 \(shadowed\)。同理，环境 E2 中变量 x 的值就是 Frame B 中的 10。

因此，每当我们谈论**变量的值**时，我们实际上再谈论的是，**在当前环境下变量的值**，例如在 scheme 解释器中，我们谈论的变量实际上就是在全局环境下该变量的值。

#### define-rule

define 规则会在当前环境的第一个 Frame 中创建，或者覆盖已有的 binding。如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.46.12 PM.jpg)

假设我们在 GE 中执行 \(define z 20\)，这时候，由于 GE 中已经含有 z 变量与 10 的 binding，因此该语句将用 z 变量与 20 的新 binding 覆盖原 binding。

假设我们在 E1 中执行 \(define z 25\)，这时候，由于 Frame A 是 E1 中的第一个 Frame，且由于 E1 中并不含有 z 变量与其它值的 binding，因此该语句将在 Frame A 中创建 z 和 25 的binding。

修改结果如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.46.52 PM.jpg)

#### set!-rule

set! 规则在执行时，首先顺着 Frame 链寻找第一个含有目标变量 binding 的 Frame，然后将那个 binding 中的数据修改为参数指定的值。如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.47.22 PM.jpg)

假设我们在 GE 中执行 \(set! z 20\)，我们从 Frame B 开始寻找，恰好 Frame B 含有 z 与 10 的 binding，因此该语句将把 z 与 10 的 binding 修改为 z 与 20 的 binding。此时 set! 与 define 的结果一致。

假设我们在 E1 中执行 \(set! z 20\)，我们从 Frame A 开始寻找，Frame A 中不含有 z 与值的 binding，因此继续寻找，直到发现 Frame B 中含有 z 与 10 的 binding，于是该语句将把 z 与 10 的 binding 修改为 z 与 20 的 binding。此时 set! 与 define 的结果就不一样。

#### lambda-rule

在替代模型中，我们规定 lambda 表达式的返回值是一个复合程序 \(compound procedure\)，但我们没有讨论这个复合程序的内容。实际上，它由两个指针构成，第一个指针指向程序的参数和程序体，另一个指针指向这个 lambda 表达式所在的环境。如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.48.13 PM.jpg)

假如我们在 E1 中执行 \(define square \(lambda \(x\) \(\* x x\)\)\) \| E1，lambda 表达式将返回复合程序，它与 y 在 Frame A 中形成一个 binding。而这个复合程序有两个指针，它的第一个指针指向它的参数和程序体，第二个指针指向它被创建的环境 E1。![](/assets/Screen Shot 2018-03-01 at 10.48.51 PM.jpg)

#### application

在环境模型中，执行符合程序 P 可以分为四个步骤：

1. 创建一个新的 Frame，设为 A
2. 将 A 的外环境指针指向 P 的外环境指针所指的环境，此时 A 及其外环境一起构成新的环境，设为 E
3. 在 A 中，将 P 的参数与它的值绑定起来
4. 在 E 中执行 P 的程序体

##### 例1：

```scheme
(define (square x) (* x x))
(square 4)
```

假设我们在 GE 中 define 了 square，那么执行 \(square 4\) 的过程如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.51.06 PM.jpg)

首先创建新的 Frame， A；因为 square 的外环境指针指向 GE，因此 A 的外环境指针也指向 GE，由图中蓝线所示；接着在 A 中添加 x 与 4 的 binding；最后在 E1 中执行程序体 \(\* x x\)，程序体中的 \* 在 GE 中有 binding 值，而 x 在 A 中有绑定值 4，最终 \(square 4\) 语句的执行结果为 16。

##### 例2：

```scheme
(define (square x) (* x x))
(define (inc-square y) (+ 1 (square y))

(inc-square 4)
```

假设我们在 GE 中 define 了 square 和 inc-square，那么执行 \(inc-square 4\) 的过程如下图所示：![](/assets/Screen Shot 2018-03-01 at 10.51.27 PM.jpg)

首先创建新的 Frame，由于 inc-square 的外环境指针指向 GE，因此新 Frame 的外环境指针也指向 GE，新 Frame 与 GE 构成新环境 E1；接着在新 Frame 中添加 y 与 4 的 binding；最后在 E1 中执行程序体 \(+ 1 \(square y\)\)，程序体中的 + 和 square 在 GE 中有 binding 值，而 y 在新 Frame 中有绑定值。但 square 对应的是一个复合程序，因此将重复例1中的过程，直到返回 16 位置，然后在 E1 中执行 \(+ 1 16\) 得到最终值 17。值得注意的是，E2 的外环境指针指向的是 GE 而非 E1，原因在于定义 square 的时候，square 复合程序的外环境指针指向的是 GE。

##### 例3：make-counter

现在我们已经拥有足够强大的模型来解释本节开头的例子 --- make-counter。这里只上图，留作意淫：![](/assets/Screen Shot 2018-03-01 at 10.51.57 PM.jpg)

### 小结

个人觉得，环境模型已经与我们现实中各门高级程序语言的模型非常接近，对于了解各门语言的执行模型来说，环境模式是一个不错的起点。

#### 参考

* [Youtube: SICP-2004-Lecture-13](https://www.youtube.com/watch?v=SDsxFreEYsc&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=13&t=0s)

* [MIT6.006-SICP-2005-lecture-notes-13](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture15webha2.pdf)



