# 第十五课 - 面向对象编程\(2\) - Scheme 实现\(1\)

### 父类和子类

上一课，我们通过下表，引申出通过对同一种数据类型的不同操作进行抽象而得到面向对象编程的思路。

|  | Data Type 1 | Data Type 2 | Data Type 3 | Data Type 4 |
| :--- | :--- | :--- | :--- | :--- |
| Operation 1 | some proc | some proc | some proc | some proc |
| Operation 2 | some proc | some proc | some proc | some proc |
| Operation 3 | some proc | some proc | some proc | some proc |
| Operation 4 | some proc | some proc | some proc | some proc |

再次分析这张表：

* 横向：同一个过程常常可以对多个数据类型进行操作，以此为出发点形成过程抽象、数据抽象的思路
* 纵向：同一个数据类型常常可以适应多种过程，以此为出发点形成面向对象的思路

但不论以哪一点为出发点，都需要兼顾另一个出发点背后隐藏的抽象需求。从面向对象的角度出发，我们将同一个数据类型的内部状态和过程结合解决纵向的抽象合并，但不同的数据类型可能含有类似的过程，这时就可以通过将这些类似的过程单独抽象成一个数据类型的方式解决，也就是我们常说的继承、is-a 关系、父类和子类。

例如：汽车和电动车的内部状态都包含位置 \(position\)，控制内部状态转变的过程都有行驶 \(drive\)，因此可以抽象出一个车 \(Car\) 类，让汽车和电动车都继承它的内部状态和相关过程，这样就能在一定程度上解决上表中横向的抽象合并。

一个支持继承的面向对象编程系统，至少需要考虑以下这些问题：

* 每个实例都有标签来表示它属于哪个类
* 声明类之间的继承关系
* 子类继承父类的状态和过程
* 如果子类没有相关过程，是否委托 \(delegation\) 给父类

### 如何用 Scheme 构建面向对象系统

Scheme 的环境模型，可以用来构建 Object 和 Class：

* Objects: 接收消息的，含有内部状态的过程集合
  * 每个实例都有 identity：唯一的 Scheme procedure
  * 每个示例都有局部状态：每个示例 procedure 都有它的局部环境
* Classes: Scheme make-&lt;object&gt; procedures:
  * 方法：接收消息，执行对应的 Scheme procedure
  * 继承规则：在继承链上如何决定调用哪个 procedure

让我们把自己的双手搞脏吧！尝试实现以下的对象系统：

![](/assets/Screen Shot 2018-03-13 at 10.48.15 PM.jpg)

#### Person

person 实例在接收到消息后，可能会需要做很多种事情，如返回信息 \(selector, predicator\)、改变内部状态，这些事情中，有些还需要输入来自外界的信息 \(参数\)，因此我们约定接收消息后，都统一返回一个 procedure，这个 procedure 可以接受参数也可以不接受参数。该约定是这个面向对象系统的一个设计，不同的面向对象系统在这类设计上可以有不同的取舍。

接下来我们实现 make-person procedure

```scheme
(define (make-person fname lname)
    (lambda (message)
        (cond ((eq? message 'WHOAREYOU?) (lambda () fname))
              ((eq? message 'CHANGE-MY-NAME)
               (lambda (new-name) (set! fname new-name)))
              ((eq? message 'SAY)
               (lambda (list-of-stuff)
                 (display-message list-of-stuff)
                 'NUF-SAID))
              (else (no-method)))))
```

利用环境模型，我们将局部状态 fname, lname 保存在局部环境中，然后利用 procedure 的外环境来访问、修改局部状态，实现之前对象系统的需求。有了 make-person，我们就可以用它创建 person 对象的实例：

```scheme
(define g (make-person 'george 'orwell))
((g 'WHOAREYOU?)) ; (g 'WHOAREYOU?) 获取方法后再执行它，才能输出 george
==> george
```

注意上面的信息传递到得到最终结果的过程，实际上包含两个步骤：

* 从实例中找到 msg 对应的 procedure
* 合理地执行这个 procedure 来获取预期结果

我们可以改进这个过程：

* 将两个过程分开
* 将两个过程对使用者抽象成一个步骤

```scheme
(define (get-method message object)
    (object message))

(define (ask object message . args)
    (let ((method (get-method message object)))
        (if (method? method)
            (apply method args)
            (error "No method for message" message))))
```

于是，刚才的调用过程就变成：

```scheme
(define g (make-person 'george 'orwell))
(ask g 'WHOAREYOU?)
==> george
```

具体的环境模型如下图所示：![](/assets/Screen Shot 2018-03-13 at 10.48.45 PM.jpg)

#### no-method 与 method?

在顺着继承关系寻找对应 procedure 的过程中，我们需要两个助手

* no-method --- 由于我们约定，每当我们向 object 索取信息时，它总是返回一个 procedure，因此我们需要一个 procedure 来表示object 内部没有相应的 procedure, 这就是 no-method

```scheme
(define no-method
  (let ((tag (list 'NO-METHOD)))
       (lambda () tag))
```

* method? --- 确认返回值是否是有效的 procedure

```scheme
(define (method? x)
  (cond ((procedure? x) #T)
        ((eq? x (no-method) #F)
        (else
          (error "Object returned non-message" x))))
```

#### 参考

* [Youtube: SICP-2004-Lecture-15](https://www.youtube.com/watch?v=2G5Yg-sOe9Q&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=15&t=0s)

* [MIT6.006-SICP-2005-lecture-notes-15](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture17_webhan.pdf)



