# 第十课 - 数据驱动编程 \(Data Directed\) 与防御式编程\(Defensive\)

### Symbol 与 Tagged Data

Tagged Data，一言以蔽之，就是为每种数据结构增加一个唯一地标识符。有了唯一标识符，写程序时就可以根据标识符来判断需要完成的相应处理，在数据结构的使用与实现解耦的基础上，增加了代码整体的易读性。

##### 举例：复数

复数在笛卡尔坐标系上可以拆分成实部 \(Real Part\) 和虚部 \(Imaginary Part\)，在极坐标系上可以拆分成模 \(Magnititue\) 和辐角 \(Angle\)。复数的操作有些在笛卡尔坐标系上比较方便，如复数的加法，只需要把两个复数的实部和虚部分别相加；而有些则在极坐标系上比较方便，如复数的乘法，只需要把两个复数的模相乘，辐角相加。假设“复数”这个数据结构的实现已经存在，我们便可以实现复数加法和乘法：

```scheme
(define (+c z1 z2)
  (make-complex-from-rect (+ (real z1) (real z2))
                          (+ (imag z1) (imag z2))))

(define (*c z2 z2)
  (make-complex-from-polar (* (mag z1) (mag z2))
                           (+ (angle z1) (angle z2))))
```

那么，数据结构“复数”会如何实现？

_**小明的实现**_

小明是传统的极简主义者，他选择用 list 来存储复数的实部和虚部。因此他的复数构造器实现如下：

```scheme
; constructors
(define (make-complex-from-rect rl im) (list rl im))
(define (make-complex-from-polar mg an)
  (list (* mg (cos an))
        (* mg (sin an))))
```

由于小明的复数内部利用实部和虚部表示，当构造器接受模和辐角时，构造器需要将其转化成实部和虚部。然后是选择器的实现，只要保证实现和使用质检的契约不变即可：

```scheme
; selectors
(define (real cx) (car cx))
(define (imag cx) (cadr cx))
(define (mag cx)
  (sqrt (+ (square (real cx))
           (square (imag cx)))))
(define (angle cx) 
  (atan (imag cx) (real cx)))
```

_**小红的实现**_

小明的同学小红，是个喜欢北极的极简主义者，她选择用 list 来存储复数的模和辐角。她的实现如下：

```scheme
; constructors
(define (make-complex-from-real rl im)
  (list (sqrt (+ (square rl) (square im)))
        (atan im rl)))
(define (make-complex-from-polar mg an) (list mg an))

; selectors
(define (real cx) (* (mag cx) (cos (angle cx))))
(define (imag cx) (* (mag cx) (sin (angle cx))))
(define (mag cx) (car cx))
(define (angle cx) (cadr cx))
```

_**问题来了**_

老师说，小明和小红需要共同完成一个关于复数应用的项目，小明或者小红遇到了**\(list a b\)**时，**a**和**b**分别表示什么？是实部和虚部还是模和辐角？显而易见，我们需要某种方式去判断这个复数来自于小明的实现还是小红的实现；或者从根本上，我们需要知道这个复数内部是用笛卡尔坐标系表示还是极坐标系表示。办法很简单，利用原始类型 symbol 为两种表示法打上标签，即**tagged complex number**。此时，构造器改造如下：

```scheme
; constructors
(define (make-complex-from-rect rl im)
  (list 'rect rl im))
(define (make-complex-from-polar mg an)
  (list 'polar mg an))

; selectors for tag and contents
(define (tag obj) (car obj))
(define (contents obj) (cdr obj))

; selectors for real part
(define (real sz)
  (cond ((eq? (tag z) 'rect) (car (contents z)))
        ((eq? (tag z) 'polar) (* (car (contents z))
                                 (cos (cadr (contents z)))))
        (else (error "unknown form of object"))))
; ...other selectors
```

### 数据驱动编程与防御式编程

从上面的实例可以看出，Tagged Data 有两个特点

* 为每种复杂数据添加标识符

* 对复杂数据操作时，根据它的标识符来判断使用哪种方法

使用 Tagged Data 变成可以让我们做两件事情：Data Directed Programming 和 Defensive Programming

_**Data Directed Programming**_

数据定向编程其实就是上文实现 “复数” 时所使用的方法，每种复杂数据自带标签，程序通过读取其身上的标签来判断应该对它做怎样的操作。这种编程方式使得模块化变得十分自然，而代码整体也变得更容易扩展和维护。

_**Defensive Programming**_

防御性编程，一言以蔽之，就是：

> It's much better to give an error message than to return garbage

换句话说，就是我们只对系统的输入中符合前提假设的进行处理，一旦遇到与假设不合的情况，立即优雅地抛错，而不是让错误的结果继续在程序里传播。

#### 例：Arithmetic Expressions

在本例中，我们将构建一个可以创建、评价数学表达式的系统，这个系统不仅可以评价简单的数学表达式，如：

```scheme
(define exp1 (make-sum (make-sum 3 15) 20))
exp1 		==> (+ (+ 3 15) 20)
(eval-1 exp1) 	==> 38
```

它还能够简化区间和精确范围，如

```
# 区间
[3, 7] + [1, 3] = [4, 10]
# 精确范围
(100±1) + (3±0.5) = (103±1.5)
```

我们从简单的情况开始：

> It is almost always easier to extend a base system, than to try to do the whole thing at once

```scheme
; ADT for sums
; type: Exp, Exp -> SumExp
(define (make-sum addend augend)
  (list '+ addend augend))

; type: anytype -> boolean
(define (sum-exp? e)
  (and (pair? e) (eq? (car e) '+)))

; type: SumExp -> Exp
(define (sum-addend sum) (cadr sum))
(define (sum-augend sum) (caddr sum))
```

此时，我们的第一版 eval 如下

```scheme
; Eval for numbers only
(define (eval-1 exp)
  (cond
   ((number? exp) exp)
   ((sum-exp? exp)
    (+ (eval-1 (sum-addend exp))
       (eval-1 (sum-augend exp))))
   (else
    (error "unknown expression" exp))))
```

然后考虑扩展到区间的 ADT

```scheme
; constructor
; type: number, number -> range2
(define (make-range-2 min max) (list min max))

; selectors
; type: range2 -> number
(define (range-min-2 range) (car range))
(define (range-max-2 range) (cadr range))

; type: range2, range2 -> range2
(define (range-add-2 r1 r2)
  (make-range-2
   (+ (range-min-2 rl) (range-min-2 r2))
   (+ (range-max-2 r1) (range-max-2 r2)))
```

让我们的 eval 兼容区间的加法

```scheme
(define (eval-2 exp)
  (cond
   ((number? exp) exp)
   ((sum-exp? exp)
    (let ((v1 (eval-2 (sum-addend exp)))
          (v2 (eval-2 (sum-augend exp))))
      (if (and (number? v1) (number? v2))
          (+ v1 v2)
          (range-add-2 v1 v2))))
   ((pair? exp) exp)
   (else (error "unknown expression" exp))))
```

然而，以上的实现有两个问题：

1. 如果 eval 数值与区间的和，系统会抛错，系统本身并未考虑数值与区间相加的情况

2. 如果我们把精确度的数据作为参数传入，按照目前的逻辑，只要不是数值就认为它是区间，因此这种实现无法做到 defensive programming

以上的例子恰好展示了我们在构建复杂系统时出现函数传入错误参数的**成因**

* 写代码手滑打错

* 逻辑有缺陷

* 改变了系统的一部分代码，但没有修改与之关联的其余代码

以及**后果**

* Garbage in garbage out

* 没有 defensive programming 导致错误在程序中传播

但究其根本原因，在于我们依赖于数据的内部结构来判断这个数据的类型，但这违背了抽象的原则，使得数据的实现和使用耦合，且数据的内部结构并不能唯一代表该数据的类型 — 因此我们需要引入**Tagged Data**, 同时引入**Data Directed Programming**与**Defensive Programming**。

##### 引入 Tagged Data 来解决上述问题

```scheme
;; SumExp

(define sum-tag '+)

; Type: Exp, Exp -> SumExp
(define (make-sum addend augend)
  (list sum-tag addend augend))

; Type: anytype -> boolean
; 此时, sum-exp? 只会识别 make-sum 创建的 SumExp
(define (sum-exp? e)
  (and (pair? e) (eq? (car e) sum-tag)))

;; ADT for numbers
(define constant-tag 'const)

; type: number -> ConstantExp
(define (make-constant val)
  (list constant-tag val))

; type: anytype -> boolean
(define (constant-exp? e)
  (and (pair? e) (eq? (car e) constant-tag)))

; type: ConstantExp -> number
(define (constant-val const) (cadr const))

; Eval for numbers with tags - eval-3
; type: ConstantExp | SumExp -> number
(define (eval-3 exp)
  (cond
   ((constant-exp? exp) (constant-val exp))
   ((sum-exp? exp)
    (+ (eval-3 (sum-addend exp))
       (eval-3 (sum-augend exp))))
   (else (error "unkown expr type: " exp))))

; 然而我们希望 SumExp -> ConstantExp, 以便后续计算
; eval-4
; type: ConstantExp | SumExp -> ConstantExp
(define (eval-4 exp)
  (cond
   ((constant-exp? exp) exp)
   ((sum-exp? exp)
    (make-constant
     (+ (constant-val (eval-4 (sum-addend exp)))
        (constant-val (eval-4 (sum-augend exp)))
     )))
   (else (error "unkown expr type: " exp))))

; 我们可以把 add 操作提取到 Constant 的 ADT 中，进一步简化代码
(define (constant-add c1 c2)
  (make-constant (+ (constant-val c1)
                    (constant-val c2))))
; eval-4
; type: ConstantExp | SumExp -> ConstantExp
(define (eval-4 exp)
  (cond
   ((constant-exp? exp) exp)
   ((sum-exp? exp)
    (constant-add (eval-4 (sum-addend exp))
                  (eval-4 (sum-augend exp))))
   (else (error "unkown expr type: " exp))))
```

从 eval-3 与 eval-4 我们可以总结出

* 带标签的 ADT 的一般模式通常由以下几点构成

  * 一个储存**标签**的变量

  * 在构造器 \(constructor\) 中将**标签**打在每个数据的 car 上

  * 写一个函数 \(predicate\) 来判断数据类型是否与**标签**一致

  * 任意操作 \(operations\) 拿到数据后，先摘下标签，再操作数据，最后将便签打上

* 使用带标签的 ADT 时

  * 要尽可能使用数据标签来判断作何操作

  * 返回时要返回带标签的数据

##### 加入区间 ADT

```scheme
; range ADT with tags
(define range-tag 'range)

; constructor
; type: number, number -> RangeExp
(define (make-range min max)
  (list range-tag min max))

; predicate
; type: anytype -> boolean
(define (range-exp? e)
  (and (pair? e) (eq? (car e) range-tag)))

; selectors
; type: RangeExp -> number
(define (range-min range) (cadr range))
(define (range-max range) (caddr range))

; eval-5
; ConstantExp | RangeExp | SumExp
; 		-> ConstantExp | RangeExp
; return ConstantExp if given constants
; return RangeExp if given combination of constants and ranges
(define (eval-5 exp)
  (cond
   ((constant-exp? exp) exp)
   ((range-exp? exp) exp)
   ((sum-exp? exp)
    (let ((v1 (eval-5 (sum-addend exp)))
          (v2 (eval-5 (sum-augend exp))))
      (if (and (constant-exp? v1) (constant-exp? v2))
          (constant-add v1 v2)
          (range-add (val2range v1) (val2range v2)))))
   (else (error "unkown expr type: " exp))))

; 用数据导向编程抽象出 add 函数
; ValueExp = ConstantExp | RangeExp
(define (value-exp? v)
  (or (constant-exp? v) (range-exp? v)))

; type: ValueExp, ValueExp -> ValueExp
(define (value-add-6 v1 v2)
  (if (and (constant-exp? v1) (constant-exp? v2))
      (constant-add v1 v2)
      (range-add (val2range v1) (val2range v2))))

; val2range: if argument is a range, return it
; else make the range [x x] from a constant x
; ValueExp = ConstantExp | RangeExp
; type: ValueExp | SumExp -> ValueExp
(define (eval-6 exp)
  (cond
   ((value-exp? exp) exp)
   ((sum-exp? exp)
    (value-add-6 (eval-6 (sum-addend exp))
                 (eval-6 (sum-augend exp))))
   (else (error "unkown expr type: " expr))))
```

如此一来，利用 Tagged Data 我们成功利用 tag 来引导程序运行以及在遇到预期之外的输入时抛错，做到了**简单**、**安全。**

##### 加入精确度 ADT

```scheme
; tag
(define limited-tag 'limited)

; constructor
; type: number, number -> LimitedExp
(define (make-limited-precision val err)
  (list limited-tag val err))

; value-add-7
; type ValueExp, ValueExp -> ValueExp
(define (value-add-7 v1 v2)
  (cond
   ((and (constant-exp? v1) (constant-exp? v2))
    (constant-add v1 v2))
   ((and (value-exp? v1) (value-exp? v2))
    (range-add (val2range v1) (val2range v2)))
   (else
    (error "unkown exp: " v1 "or " v2))))

; eval-7
; type: ValueExp|LimitedExp|SumExp -> ValueExp|Limited
(define (eval-7 exp)
  (cond
   ((value-exp? exp) exp)
   ((limited-exp? exp) exp)
   ((sum-exp? exp)
    (value-add-6 (eval-7 (sum-addend exp))
                 (eval-7 (sum-augend exp))))
   (else (error "unkown expr type: " exp))))
```

从 eval-5 到 eval-7 我们可以总结出：

* Data Directed Programming 可以简化逻辑层次较高的代码

* 程序在每次操作前都检查了标签才能真正做到 Defensive Programming

* 通常情况下，ADT 内部不会检查**标签**，原因在于性能考虑

### 参考

* [Youtube: SICP-2004-Lecture-10](https://www.youtube.com/watch?v=RI8yXdJ2N1E&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=10)

* [MIT6.006-SICP-2005-lecture-notes-10](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture10webhan.pdf)



