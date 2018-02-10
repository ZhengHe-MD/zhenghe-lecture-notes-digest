# 第五课 - 数据抽象

### 引入

本课之前，我们主要介绍程序抽象，即用一段程序将平时计算中的频繁重复出现的模式 \(pattern\) 记录起来 \(通过 define 和 lambda 表达式\)，之后每次使用只需要调用这段程序就能重现 \(通过 define 的 name\)。这么做可以将这个通用计算模式的实现与使用隔离开来。第五课主要介绍另一种抽象 — 数据抽象，它将类型不同、内容不同的数据以及操纵这些数据的方法组合，当作系统中的原始数据类型使用。讲稿中有一句话值得记录

> The whole goal of a high-level language is to allow us to suppress unneccesary detail in this manner, while focusing on the use of a procedural abstraction to support some more complex computational design.

### 程序抽象回顾

当我们把一个计算模式抽象成一段程序后，就把该计算模式的使用与实现隔离。对程序的使用者来说，程序就是一个黑箱，但这个黑箱会遵守一种协议，这个协议规定了输入到输出的映射。

![](/assets/mit-6.001-5-procedural-abstraction.jpg)以牛顿法求平方根为例：

```scheme
(define try (lambda (guess x)
              (if (good-enuf? guess x)
                  guess
                  (try (improve guess x) x))))
(define improve (lambda (guess x)
                  (average guess (/ x guess))))
(define average (lambda (a b) (/ (+ a b) 2)))
(define good-enuf? (lambda (guess x)
                     (< (abs (- (square guess) x)) 0.001)))
(define sqrt (lambda (x) (try 1 x)))
```

那么如果把这个黑箱的内部也展示出来，可以得到下图：![](/assets/mit-6.001-5-sqrt-white-box.jpg)但仔细思考可以发现 average 是一种更为通用的计算模式，并不是为求平方根专门设计，因此我们可以将 sqrt 内部的几个计算模式划分为通用和专用两种，average 属于通用，而 improve, good-enuf?, try, sqrt 则属于专用：

```scheme
(define average (lambda (a b) (/ (+ a b) 2)))
(define sqrt
  (lambda (x)
    (define good-enuf? (lambda (guess)
                         (< (abs (- (sqare guess) x) 0.001))))
    (define improve (lambda (guess)
                      (average guess (/ x guess))))
    (define sqrt-iter (lambda (guess)
                        (if (good-enuf? guess) 
                            guess
                            (sqrt-iter (improve guess)))))
    (sqrt-iter 1.0)))
```

### 从复合数据 \(Compound Data\) 到数据抽象 \(Data Abstraction\)

现实世界中的概念复杂程度各有不同。简单的概念可以用原始数据类型来表示，比如身高、体重、距离、名字、命题真假；而复杂的概念则需要通过多个多种原始数据类型组合来表示，比如人的体检单会有身高、体重、姓名、以及其它各项指标。当一个概念不能用单个原始数据表示时，我们就需要将原始数据组合成复合数据来表示。当有了体检单这种复合数据后，我们还需要有将复合数据拆分成其组成部分的能力。这种组合 \(glue\) 和拆分 \(unglue\) 构成了复合数据的协议 \(contract\)。

以 Scheme 中的 pair 为例，它的协议如下：

* cons 将两个元素组合起来

* car 取出第一个元素

* cdr 取出第二个元素

pair 可以被当做原始数据类型一样对待，可以被用作更复杂的复合数据的组成元素，再举一例：

```scheme
; 利用 cons 来组合二维空间中的点 (x, y)，并定义拆分方式 point-x 和 point-y
(define (make-point x y)
  (cons x y))
(define (point-x point)
  (car point))
(define (point-y point)
  (cdr point))
; 有了点，我们就可以构造线
(define (make-seg pt1 pt2)
  (cons pt1 pt2))
(define (start-point seg)
  (car seg))
(define (end-point seg)
  (cdr seg))
```

**复合数据与建立其上的协议共同构成了数据抽象**，以 pair 抽象为例，这种抽象通常有以下三个标准组成部分：

```
;; Constructor
; cons: A,B -> Pair<A,B>
  (cons <x> <y>) => <P>
  
;; Accessors
; car: Pair<A,B> -> A
  (car <P>) => <x>
; cdr: Pair<A,B> -> B
  (cdr <P>) => <y>
  
;; Predicate
; pair? anytype -> boolean
  (pair? <x>) => #t if <x> evaluates to a pair, else #f
```

值得一提的是，当有了 Constructor 和 Accessors 以后，我们可以忘记复合数据背后是如何运作的，只需要按着协议使用即可，而只要协议被遵守，任意修改背后的运作实现也不会影响到使用者已有代码。

pair 抽象是组合两部分数据的好方式，但有时候，我们需要将多部分数据组合在一起，这时候就要借助 list。以下是 list 抽象：

```scheme
; list 由多个 pair 连接而成，且最后一个元素是 empty list --- 即 ()

;; Constructor
; list: el1, el2, ... -> List<el1, el2, ...>
; (list el1 el2 ...) => <L>

;; Accessor
(define first car) ; first 返回 list 中的第一个元素
(define rest cdr)  ; rest 返回 list 中剩下的元素组成的 list
; first: List<el1, el2...> -> el1
; (first <L>) => <el1>
; rest:  List<el1, el2...> -> List<el2, el3,...>
; (rest <L>) => List<el2, el3, ...>

;; Predicate
; null? anytype -> boolean
; (null? <x>) => #f if <x> evaluates to an empty list, else #f
```

建立数据抽象的目的之一在于这种抽象能使得一些操作变得简便，这些操作通常是数据抽象的第四个组成部分：

```scheme
;; Operations
; Common Pattern #1: cons'ing up a list --- 举例：在区间中生成步长为 1 的 list
(define (enumerate-interval from to)
  (if (> from to)
      nil
      (adjoin from
              (enumerate-interval (+ 1 from) to))))
; Common Pattern #2: cdr'ing down a list  --- 举例：找到 list 中的第 n 个元素
(define (list-ref lst n)
  (if (= n 0)
      (first lst)
      (list-ref (rest lst) (- n 1))))
; Common Pattern #3: cdr'ing and cons'ing  --- 举例：复制 list 
(define (copy list)
  (if (null? 1st)
      nil
      (adjoin (first lst)
              (copy (rest lst)))))
; Common Pattern #3: cdr'ing and cons'ing  --- 举例：将两个 list 串联
(define (append list1 list2)
  (cond ((null? list1) list2)
        (else
          (adjoin (first list1)
                  (append (rest list1)
                          list2)))))
```

数据抽象，理论上应该让用户完全不需要知道底层实现就可以做到自己想要实现的事情，而数据抽象的提供者应该能够在保持 constract 不变的情况下，自由改动底层实现而不影响用户。这种使用与实现之间的壁垒成为抽象壁垒 \(Abstraction Barrier\)。

### 小结

本课目的在于理解：复合数据与建立其上的协议共同构成的数据抽象，帮助抽象数据的使用者和开发者划清界限，只要保证遵守协议，双方都可以自由地做自己的事情而不受对方影响。

### 参考

[MIT6.006-SICP-2005-lecture-notes-5](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture5webhand.pdf)



