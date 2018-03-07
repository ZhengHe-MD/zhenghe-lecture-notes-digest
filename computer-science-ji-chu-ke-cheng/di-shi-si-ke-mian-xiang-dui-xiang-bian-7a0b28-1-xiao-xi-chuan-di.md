# 面向对象编程 \(1\) - 消息传递 \(Message Passing\)

本节要点：

* 回顾前几课学到的 “抽象”
* 消息传递为基础的面向对象编程

### 抽象回顾

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
| Operation 2 | some proc | some proc  | some proc | some proc |
| Operation 3 | some proc | some proc | some proc | some proc |
| Operation 4 | some proc | some proc | some proc | some proc |

实际上，我们刚才的解决问题的思路就是将不同 Data Types 的相同的 Operation 抽象出来，也就是表中的一行；如果我们转换思路，将一种 Data Type 的不同 Operation 抽象出来，也就是表中的一列，会有怎样的变化？

### 另一种角度：数据是有内部状态的过程

在第十三课中介绍到，每个 procedure 由两个部分构成

* 参数 \(parameters\) 和过程体 \(body\)
* 环境 \(container for name-value bindings\)

利用这个模型，我们就可以尝试将数据看成是有内部状态的过程，它的内部状态在环境中，而过程的执行则在同样的环境中发生。此时，我们将数据内部状态封装 \(encapsulated\) 在过程中，对外不可见。

##### 例：Pair as a Procedure with State

```scheme
(define (cons x y)
    (lambda (msg)
        (cond ((eq? msg 'CAR) x)
              ((eq? msg 'CDR) y)
              ((eq? msg 'PAIR?) #t)
              (else (error "pair cannot" msg)))))
(define (car p) (p 'CAR))
(define (cdr p) (p 'CDR))
(define (pair? p)
    (and (procedure? p) (p 'PAIR?)))
```

上面的代码接收外部传入的 msg，利用 msg 来判断下一步所做的操作，这种编程风格被称为消息传递 \(message passing\)。消息传递编程将系统看成是多种对象以及它们之间的交流，解决问题的过程就是对象之间合理交流的结果。值得一提的是，在此之前，我们利用 Tagged Data 中的 tag 来区分不同的数据结构，这里，我们之间将 tag 转化成内部过程的一部分。
