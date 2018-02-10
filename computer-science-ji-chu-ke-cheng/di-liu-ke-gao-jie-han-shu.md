# 第六课 - 高阶函数

### Scheme 中的类型

程序中的各种结构，如**变量**、**表达式**、**函数**等都有一种属性叫做类型，而制定类型规则的系统称为类型系统 \(Type System\) 。类型系统能够通过规定程序中不同部分之间接口类型，同时检查接口的使用是否遵守既定规则，来减少 bug 出现的可能。此外，在程序编译时进行类型检查的语言称为**静态类型语言**；在程序运行时进行类型检查的语言称为**动态类型语言**。

Scheme 的基本数据类型包括：

1. Number

2. String

3. Boolean

4. Names \(symbols\)

Scheme 的复合数据类型包括：

1. Pair&lt;A, B&gt;

2. List&lt;A&gt;= Pair&lt;A, List&lt;A&gt;or nil&gt;

Scheme 的程序类型包括参数的数量、类型以及结果的类型，举例如下：

1. square: number -&gt; number

2. &gt;: number, number -&gt; boolean

3. +: number, number -&gt; number

当类型不符合程序本身的类型定义时，运行程序就会报错：

```scheme
> (+ "hello" "world")
Exception in +: "hello" is not a number
```

类型帮助我们提高思考程序的效率：

1. 减少常见 bug

2. 为算法分析与优化建立基础

### 从类型到高阶函数

#### 例1：求和

计算过程中，我们会遇到这些计算模式：![](/assets/Screen Shot 2018-02-10 at 12.09.51 PM.jpg)运用前几节课的知识，可以用程序段描述它们：

```scheme
(define (sum-integers a b)
  (if (> a b)
      0
      (+ a (sum-integers(+ a 1) b))))
(define (sum-squares a b)
  (if (> a b)
      0
      (+ (square a) (sum-squares (+ a 1) b))))
(define (pi-sum a b)
  (if (> a b)
      0
      (+ (square (/ 1 a)) (pi-sum (+ a 2) b))))
```

描述的过程中容易发现，这三段程序的计算模式非常相近，除了两个地方：

1. 每次累加的项 \(term\)：

   * sum-integers:**a**

   * sum-squares:**\(square a\)**

   * pi-sum:**\(+ \(square \(/ 1 a\)\)\)**

2. 进入下一步 \(next\):

   * sum-integers:**\(+ a 1\)**

   * sum-squares:**\(+ a 1\)**

   * pi-sum:**\(+ a 2\)**

尽管三段程序的 term 和 next 具体形式不尽相同，但不同的 term 、不同的 next 类型一致，因此我们容易构造出以下计算模式。

```scheme
(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) next b))))
; 类型：(number -> number, number, number -> number, number) -> number
```

于是，利用上述计算模式，可以得到

```scheme
(define (sum-integers a b)
  (sum (lambda (x) x) a (lambda (x) (+ x 1)) b))
(define (sum-squares a b)
  (sum square a (lambda (x) (+ x 1)) b))
(define (pi-sum a b)
  (sum (lambda (x) (/ 1 (square x))) a
       (lambda (x) (+ x 2)) b))
```

这里的 sum 就是传说中的**高阶程序 **\(higher-order procedure\)。

#### 例2：导函数

对于简单的函数，我们可以根据基本导函数的推导规则推导得到：![](/assets/Screen Shot 2018-02-10 at 12.16.57 PM.jpg)

这里的 D 实际上有如下近似公式：

![](/assets/Screen Shot 2018-02-10 at 12.17.05 PM.jpg)

用程序来描述即：

```scheme
(define deriv
  (lambda (f)
    (lambda (x) 
      (/ (- (f (+ x epsilon)) (f x)) 
         epsilon))))
```

#### 例3：list operators

##### list transformation

```scheme
; list contract
(define (adjoin ele lst2)
  (cons ele lst2))
(define (first lst)
  (car lst))
(define (rest lst)
  (cdr lst))

; list transformation
(define (square-list lst)
  (if (null? lst)
      '()
      (adjoin (square (first lst))
              (square-list (rest lst)))))

(define (double-list lst)
  (if (null? lst)
      '()
      (adjoin (* 2 (first lst))
              (double-list (rest lst)))))
```

**square-list **与 **double-list **同样结构十分相似，不同的地方在于对每个元素所做的转换。尽管这些转换的形式不同，但它们具有相同的类型，因此我们可以用 MAP 描述这种更加泛化的计算模式：

```scheme
; 这里用 MAP 而不是 map 原因在于 map 是内置函数
(define (MAP proc lst)
  (if (null? lst)
      '()
      (adjoin (proc (first lst))
              (MAP proc (rest lst)))))

(define (square-list lst)
  (MAP square lst))
(define (double-list lst)
  (MAP (lambda (x) (* 2 x)) lst))
```

##### list filtering

过滤程序的不同点在于谓词的不同，但谓词具有相同类型 \(number -&gt; Boolean\)，因此我们可以将过滤泛化。

```scheme
; list filtering
(define (filter pred lst)
  (cond ((null? lst) '())
        ((pred (first lst))
         (adjoin (first lst)
                 (filter pred (rest lst))))
        (else (filter pred (rest lst)))))
; (number -> Boolean), list -> list
(filter even? (list 1 2 3 4 5 6))
```

##### list accumulation

累加程序的不同点在于累加操作和初始值的不同，但累加操作有相同类型 \(number, number -&gt; number\)，初始值也有相同类型 \(number\)，因此我们可以将累加泛化：

```scheme
; 这里是 FOLD-RIGHT 而不是 fold-right 原因在于后者是内置函数
(define (FOLD-RIGHT op init lst)
  (if (null? lst)
      init
      (op (first lst)
          (FOLD-RIGHT op init (rest lst)))))

(define (add-up lst)
  (FOLD-RIGHT + 0 lst))
(define (mul-all lst)
  (FOLD-RIGHT * 1 lst))
```

假设我们想要计算一种特殊的累加：

![](/assets/Screen Shot 2018-02-10 at 12.22.51 PM.jpg)

```scheme
(define (generate-interval a b)
  (if (> a b)
      '()
      (cons a (generate-interval (+ 1 a) b))))

(define (sum f start inc terms)
  (FOLD-RIGHT + 0
              (MAP (lambda (x) (f (+ start (* x inc))))
                   (generate-interval 0 terms))))
```

上文中提到的特殊的累加，其实就是对积分过程的抽象描述，不论是对什么类型的函数进行积分，只要把函数传入积分过程中，描述积分过程的不需要关注传入的函数细节。这里不再援引课程中的例子。

#### 例4： 不动点

当一个函数的定义域中存在一个点，使得函数的输出点与之相同，我们称该点为不动点。计算平方根的另一个思路就是计算以下函数的不动点：

![](/assets/Screen Shot 2018-02-10 at 12.26.25 PM.jpg)找不动点的过程可以概括如下：

1. 产生一个猜测点 x

2. 计算 f\(x\)，如果不够接近 x 本身，回到第一步

```scheme
(define (close? u v) (< (abs (- u v)) 0.0001))
(define (fixed-point f i-guess)
  (define (try g)
    (if (close? (f g) g)
        (f g)
        (try (f g))))
  (try i-guess))
```

如此一来，我们可以用它来计算平方根，黄金分割点

```scheme
; 平方根
(fixed-point (lambda (x) (/ 2 x)) 1)
; 黄金分割点
(fixed-point (lambda (x) (+ 1 (/ 1 x))) 1)
```

但如果我们利用上述方式来求 2 的平方根，可以看到猜测点在 1 和 2 之间来回震荡，因此我们需要某种方式来抑制震荡，常用的一种抑制震荡方式是平均抑制震荡 \(average-damp\)：

```scheme
(define (average-damp f)
  (lambda (x)
    (average x (f x))))
; (number -> number) -> (number -> number)

(define (sqrt x)
  (fixed-point
	(average-damp
   		(lambda (y) (/ x y)))
   	1))
```

如此一来，我们将求平方根的过程拆分成了求不动点和抑制震荡两个过程，同时隐藏了抑制震荡的细节，这使得程序员更能把注意力集中到问题本身。

### 高阶函数的作用

高阶函数极大地增强了语言的表达能力，它帮助我们在描述复杂过程时隐藏不必要的细节，从而实现对更复杂系统的把控。这很像交流过程中用到专业术语，使用专业术语时，人们可以隐去对不必要的细节的描述，从而利用更精简的语句描述复杂的过程。

### 小结

从类型到高阶函数，让 Scheme 有了更强大的表达能力，帮助我们：

1. 描述复杂过程时忽略暂时不必关注的细节，专注问题本身

2. 分析进程的演化过程，确定每个计算阶段所需的输入

### 参考

* [MIT6.006-SICP-2005-lecture-notes-6](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture6webhand.pdf)

* [Youtube: SICP-2004-lecture-6](https://www.youtube.com/watch?v=2G_azOQpR3k&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=6)



