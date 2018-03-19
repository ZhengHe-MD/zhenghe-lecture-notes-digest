# 第十五课 - 面向对象编程\(3\) - Scheme 实现\(2\)

### self

很多时候，一个对象需要在一个 procedure 中调用自己的另一个 procedure，但它没有相关的 reference，这时候就需要一个变量 self，它始终指向这个对象自己，从而达到调用自己的 procedure 的目的。

首先，需要为每个 procedure 增加 self 引用，然后就可以利用这个 self 去调用自己的 procedure：

```scheme
(define (make-person fname lname)
  (lambda (message)
    (case message
      ((WHOAREYOU?) (lambda (self) fname)
      ((CHANGE-NAME)
        (lambda (self new-name)
          (set! fname new-name)
          (ask self 'SAY (list 'call 'me fname))))
      ((SAY)
        (lambda (self list-of-stuff)
          (display-message list-of-stuff)
          'NUF-SAID))
        (else (no-method)))))
```

接着，需要修改 ask

```scheme
(define (ask object message . args)
  (let ((method (get-method message object)))
    (if (method? method)
      (apply method object args)
      (error "No method for message" message))))
```

从以上代码中，可以体会到将寻找 procedure 和调用 procedure 的逻辑抽象到 ask 中，也能让我们很方便地做这种额外的改动。

### object typing

在面向对象系统中，常常需要知道某对象的类型，从而构建对不同类型对象的处理逻辑。其中最简单的一种方法就是在对象中添加一个 procedure

```scheme
(define (make-person fname lname)
  (lambda (message)
    (case message
      ((WHOAREYOU?) (lambda (self) fname))
      ((CHANGE-NAME)
        (lambda (self new-name)
          (set! fname new-name)
          (ask self 'SAY (list 'call 'me fname))))
      ((SAY)
        (lambda (self list-of-stuff)
          (display-message list-of-stuff)
          'NUF-SAID))
      ((PERSON?)
        (lambda (self) #t))
      (else (no-method)))))

(define someone (make-person 'bert 'sesame))
(ask someone 'person?)
> #t
```

这种方法简单，但弊端也很明显，如果我们 \(ask someone 'professor\)，就会得到 no-method，但我们同样可以利用类似 ask 抽象的方式解决这个问题：

```scheme
(define (is-a object type-pred)
  (if (not (procedure? Object))
      #f
      (let ((method (get-method type-pred object)))
          (if (method? Method)
              (ask object type-pred)
              #f)))))
(define someone (make-person 'bert 'sesame))
(ask someone 'professor?)
> #f
```

### Inheritance

#### Internal object

继承 \(inheritance\) 是面向对象系统中重要的一员，它可以将系统中的个体按层级抽象，将不同个体的共同特征单独抽象，使得代码往模块化更进一步。基于之前的面向对象系统设计，我们可以在子类实例内部创建一个父类的实例，当子类中找不到与 message 相对应的 procedure 时，将 message 传递给内部的父类实例，从而实现局部变量和 procedure 的继承，以 professor 和 person 为例：

```scheme
(define (make-professor fname lname)
  (let ((int-person (make-person fname lname)))
    (lambda (message)
      (case message
        ((LECTURE) ...) ; new method
        ((WHOAREYOU?
          (lambda (self)
            (display-message (list 'Professor lname))
            lname))
        (else (get-method message int-person))))))

(define e (make-professor 'eric 'grimson))
```

执行最后一句 define，我们在全局环境上创建一个 professor 实例，此时环境模型如下图所示：

（图1）

当执行 professor 特有的 procedure 时，可以得到如下环境模型图：

（图2）

当执行 person  特有的 procedure 时，可以得到如下环境模型图：

（图3）

环境模型图中展现出整个继承的过程，值得回味。

#### Delegation

拥有 internal object 能够将子类共用的局部变量和 procedure 抽象到父类中，但有时候子类的某个 procedure 常常是父类的某个 procedure 的改进版本，为了避免重复父子类中相似 procedure 中的共同逻辑，我们需要 delegation，来实现子类对父类 procedure 的调用。

首先，构建一个 delegate  procedure

```scheme
(define (delegate to from message . args)
  (let ((method (get-method message to)))
    (if (method? method)
      (apply method from args)   ; from becomes self
      (error "No method" message))))
; 对比 ask
(define (ask object message . args)
  (let ((method (get-method message object)))
    (if (method? method)
      (apply method object args) ; object becomes self
      (error "No method for message" message)
```

delegate 与 ask 非常相似，唯一的不同在于 delegate 是从 to 身上找到 method，然后执行的时候用 from 当作 self 传入，举例：子类实例拿父类的方法应用到自己身上，而不是父类的示例身上。

继续之前的例子，创建一个 arrogant-professor，它的 SAY procedure 会在自己说的每句话之后加上 obviously，利用 delegate 实现如下：

```scheme
(define (make-arrogant-professor fname lname)      ; subclass
  (let ((int-prof (make-professor fname lname)))   ; superclass
    (lambda (message)
      ((SAY)
        (lambda (self stuff)
          (delegate int-prof self
            'SAY (append stuff '(obviously)))))
      (else
        (get-method message int-prof))))))

(define e (make-arrogant-professor 'big 'gun))
(ask e 'SAY '(the sky is blue))
> the sky is blue obviously
(ask e 'LECTURE '(the sky is blue))
> therefore the sky is blue
```

调用 SAY 时，arrogant-professor 实例如愿在自己说的话后面加上 obviously，然而调用 professor 实例的 LECTURE 时，并没有如愿。仔细看一下 make-professor 的源码：

```scheme
(define (make-professor name)
  (let ((int-person (make-person name)))
    (lambda (message)
      (case message
        ((LECTURE)
          (lambda (self stuff)
; bug       (delegate int-person self 'SAY
; bug         (append '(therefore) stuff))
            (ask self 'SAY
              (append '(therefore) stuff))))
      (else (get-method message int-person))))))
      
```

原因在于arrogant professor 内部的 professor 实例内部调用 SAY 时，使用的并不是 arrogant-professor 本身的 SAY，而是 professor 实例内的 SAY，因此 obviously 没有被加在每句话之后。因此稍加改动就能实现我们最初的目的。本例也能体会出，在面向对象系统设计过程中，在重用 procedure 过程中的一些微妙的变化。

