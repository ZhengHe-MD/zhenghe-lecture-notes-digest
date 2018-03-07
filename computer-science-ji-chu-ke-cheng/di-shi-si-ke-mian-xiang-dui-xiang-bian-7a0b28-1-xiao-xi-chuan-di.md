# 面向对象编程 \(1\) - 消息传递 \(Message Passing\)

本节要点：

* 回顾前几课学到的 “抽象”
* 消息传递为基础的面向对象编程

### 回顾抽象

在前几课，我们学过的抽象主要包括两种：

* 过程抽象 \(procedural abstractions\)
* 数据抽象 \(data abstractions\)

##### 过程抽象

主要指将一段常见的计算模式抽象成一个过程 \(procedure\)，并赋予这个抽象的过程一个名字。使用时，直接通过该名字执行这个过程，无须了解执行的具体细节，从而将这个过程的使用和实现细节分隔开。这种抽象适用于容易用函数式编程方式解决的问题，把问题的解决过程看作是一个流水线，每一个环节将给定的输入转化为确定的输出，没有副作用。

##### 数据抽象

主要指将现实问题中的主要信息抽象成一定的数据结构 \(data structure\)，同时提供构造器、选择器等一些标准接口，使得这些信息的表示细节、存储方式以及一些标准操作的实现与提供给使用者的标准接口隔离开。这意味着使用者无需了解数据结构的内部细节实现就可以利用它的标准接口来解决现实问题。

之前，在解决问题的过程中，我们通常会同时使用过程抽象和数据抽象来解决问题，数据抽象的标准接口实际上就是抽象过程的名字。本课的初衷是介绍如何解决**大型系统复杂性控制**的问题，而抽象正是解决这个问题的核心思想，如果可以不断地将实现和细节隔离开，就可以通过抽象层次的增加来达到控制复杂性的目的。

##### 利用过程、数据抽象解决问题的过程

从第十课开始，我们在一般的抽象数据中加入标识符来分辨抽象数据类型，构造 Tagged Data，然后进行数据驱动编程以及防御式编程。在一个复杂系统中，不同的抽象数据类型常常会有相同的操作，我们利用过程抽象将对不同抽象数据类型的相同操作进一步抽象，得到如下示例的抽象过程：

```scheme
(define (scale x factor)
    (cond ((number? x) (* x factor))
          ((line? x) (line-scale x factor))
          ((shape? x) (shape-scale x factor))
          (else (error "unknown type")))))

(define (translate x delta)
    (cond ((number? x) (+ x delta))
          ((line? x) (line-translate x delta))
          ((shape? x) (shape-translate x delta))
          (else (error "unknown type")))))
```

此时，系统在扩展的时候，有两种可能：

* 新增抽象数据类型
  * 需要修改所有已有的抽象过程
  * 新增的抽象数据类型名称与已有的类型名称不可重复
* 新增抽象过程
  * 创建抽象过程

这种解决问题的方式适用于有以下特点的系统：

* 抽象数据类型少且稳定
* 主要扩展发生在抽象过程上
* 不同的抽象数据类型之间几乎相互独立

**但并非所有问题都适用于这种方法，那么我们还有什么别的抽象方法呢？**

让我们用一个比较广的视角来看：一个复杂系统由系统中的数据个体以及对这些数据个体的操作所构成，于是一个复杂系统的组成部分就可以用一张二维的表格来表示：

|  | Data Type 1 | Data Type 2 | Data Type 3 | Data Type 4 |
| :--- | :--- | :--- | :--- | :--- |
| Operation 1 | some proc | some proc | some proc | some proc |
| Operation 2 | some proc | some proc | some proc | some proc |
| Operation 3 | some proc | some proc | some proc | some proc |
| Operation 4 | some proc | some proc | some proc | some proc |

实际上，我们刚才的解决问题的思路就是将不同 Data Types 的相同的 Operation 抽象出来，也就是表中的一行；如果我们转换思路，将一种 Data Type 的不同 Operation 抽象出来，也就是表中的一列，会有怎样的变化？

### 另一种角度：数据是有内部状态的过程

在第十三课中介绍到，每个 procedure 由两个部分构成

* 参数 \(parameters\) 和过程体 \(body\)
* 环境 \(container for name-value bindings\)

利用这个模型，我们就可以尝试将数据看成是有内部状态的过程，它的内部状态在环境中，而过程的执行则在同样的环境中发生。此时，我们将数据内部状态封装 \(encapsulated\) 在过程中，对外不可见。

#### 例：Pair as a Procedure with State

```scheme
(define (cons x y)
    (lambda (msg)
        (cond ((eq? msg 'CAR) x)
              ((eq? msg 'CDR) y)
              ((eq? msg 'PAIR?) #t)
              ((eq? msg 'SET-CAR!)
                  (lambda (new-car) (set! x new-car)))
              ((eq? msg 'SET-CDR!)
                  (lambda (new-cdr) (set! y new-cdr)))
              (else (error "pair cannot" msg)))))
(define (car p) (p 'CAR))
(define (cdr p) (p 'CDR))
(define (pair? p)
    (and (procedure? p) (p 'PAIR?)))
(define (set-car! p new-car)
    ((p 'SET-CAR!) new-car))
(define (set-cdr! p new-cdr)
    ((p 'SET-CDR!) new-cdr))
```

上面的代码接收外部传入的 msg，利用 msg 来判断下一步所做的操作，这种编程风格被称为消息传递 \(message passing\)。消息传递编程将系统看成是多种对象以及它们之间的交流，解决问题的过程就是对象之间合理交流的结果。值得一提的是，在此之前，我们利用 Tagged Data 中的 tag 来区分不同的数据结构；现在，我们将 tag 转化成内部过程的一部分。

```scheme
(define foo (cons 1 2))
(car foo)
; (car foo) | GE
; (foo 'CAR) | E2
```

环境模型如下所示：

（图1）

数据内部的状态始终保持在 E1 中，foo 接收到外界的消息后，对 E1 中的 x, y 进行操作，完成 **car** 的操作。

再看 mutation 操作

```scheme
(define bar (cons 3 4))
(set-car! bar 0)
```

环境模型如下图所示，看的时候要参考环境模型中执行 procedure 的四个步骤

（图2）

mutation 操作返回的是一个 lambda 过程，通过调用这个 lambda 过程可以达到修改数据内部状态的目的。

#### 思考：

利用 message-passing 的编程方式，我们创造了一种与 scheme 自带的 cons 不同的抽象。scheme 原生的 cons 就是数据的状态，而刚刚构造的 cons 则是一个含有内部状态的过程，我们也称它为一种数据抽象。但是，mutatior 和 selector 一个返回 procedure，一个返回值，这种行为是不一致的，我们可以继续改良它。

```scheme
(define (cons x y)
    (define (change-car new-car) (set! x new-car))
    (define (change-cdr new-cdr) (set! y new-cdr))
    (lambda (msg . args)
        (cond ((eq? msg 'CAR) x)
              ((eq? msg 'CDR) y)
              ((eq? msg 'PAIR?) #t)
              ((eq? msg 'SET-CAR!)
                  (change-car (first args))
              ((eq? msg 'SET-CDR!)
                  (change-cdr (first args))
              (else (error "pair cannot" msg)))))
(define (car p) (p 'CAR))
(define (set-car! p val) (p 'SET-CAR! val))
```

利用 define 的规则，在环境内部定义私有过程 \(private procedure）使得 car 和 set-car! 的表现一致。

### 面向对象编程

我们将数据内部的状态及操作内部状态的过程合起来称为数据对象 \(data objects\)，回顾刚才的编程思路，我们不断地思考：

* 数据对象 cons 应当接收哪些 messages
* 数据对象 cons 可以支持哪些操作
* 数据对象内部应当保持哪些状态

这也就形成了面向对象编程的雏形。

面向对象与面向过程是解决复杂问题的两种思路，孰优孰劣取决于问题的特点。面向过程编程比较适合数学计算、数据转化等，数值转化多，数据种类少，数据之间关系独立的问题；面向对象编程比较适合数据种类多，数据之间关系紧密的问题。此外，很重要的一点是，面向对象与面向过程仅仅是**思路的区别，并不涉及语言本身，它们与语言无关**。

#### 参考

* [Youtube: SICP-2004-Lecture-14](https://www.youtube.com/watch?v=s2S30l6ofcE&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=14&t=0s)

* [MIT6.006-SICP-2005-lecture-notes-14](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture16webhan.pdf)



