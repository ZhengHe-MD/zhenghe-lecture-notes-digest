# 第十二课 - 数据变动

之前的课程中，我们已经讨论数据抽象的几个重要组成部分：构造器 \(constructors\)、选择器 \(selectors\)、操作 \(operations\) 以及契约 \(contract\)。只要按照数据的契约来使用、操作数据，程序就会向着我们预想中的方向运行。为了达到目的，我们需要反复从旧数据中构造新数据。要是我们能够直接修改旧数据，情况又会发生什么样的变化？本节课将引入数据变动 \(Data Mutation\) ，丰富我们的数据抽象思路。

替代模型 \(substitution model\) 假设我们使用函数式编程 \(functional programming\)，没有数据变动和副作用，每段程序都像一个数学函数，无论在什么时候，都准确地将确定的输入映射到确定的输出上。但如果我们引入数据变动，每段程序的输出结果就不仅仅只依赖于输入，还依赖于程序被执行时的环境。举例如下：

```scheme
(define x 10)

(+ x 5)
> 15

(set! x 94)

(+ x 5)
> 99
```

我们将看到

> 引入数据变动使得许多事情变得容易，但会提高犯错的可能性。

### Pair/List Mutation

除了构造器、选择器、操作外，数据抽象可能还包含修改器 \(mutators\)。将数据变动引入到 Scheme 的 pair/list 中，可以得到如下的 pair/list 数据抽象：

```scheme
；constructor
(cons x y)                     ; creates a new pair p

; selectors
(car p)                        ; returns car part of pair
(cdr p)                        ; returns cdr part of pair

; mutators
(set-car! p new-x)             ; changes car pointer in pair
(set-cdr! p new-y)             ; changes cdr pointer in pair
; Pair, anytype -> undef       ; side-effect only
```

其中，set-car! 和 set-cdr! 分别修改 pair 的 car 部分和 cdr 部分指针指向的数据，而且这两种方法只有副作用，没有返回值，因此其返回值是不确定的 \(unspecified\)。

#### 小心！

mutation 给我们编程带来方便的同时引入问题，需要我们格外注意：

```scheme
(define a (list 1 2))
(define b a)

; a ==> (1 2)
; b ==> (1 2)

(set-car! a 10)
; a ==> (10 2)
; b ==> (10 2)
```

由于 a 和 b 指向同一块内存，修改 a 的同时，也会修改 b。因此我们需要在充分了解 mutation 的基础上加以使用。

