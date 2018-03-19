# 第十五课 - 面向对象编程\(3\) - Scheme 实现\(2\)

### self

很多时候，一个 object 需要在一个 procedure 中调用自己的另一个 procedure，但它没有相关的 reference，这时候就需要一个变量 self，它始终指向这个 object 自己，从而达到调用自己的 procedure 的目的。

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



