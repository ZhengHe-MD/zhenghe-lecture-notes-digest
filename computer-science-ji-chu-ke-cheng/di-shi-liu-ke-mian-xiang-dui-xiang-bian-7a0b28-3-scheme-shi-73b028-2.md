# 第十五课 - 面向对象编程\(3\) - Scheme 实现\(2\)

#### Detection of methods

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



