# 第十九课 - Normal Order \(Lazy\) Evaluation

### Applicative Order & Normal Order

之前，我们已经了解了一个 Scheme 解释器的重要组件以及组件之间的关系。当我们已经熟悉的 Scheme evaluator 接受到一个 application expression时，它会首先 evaluate application 中的 sub-expression，再把各个 sub-expression 的结果传入 application 表达式的 procedure，进而执行这个 procedure 得到最终结果，这种 evaluation 的方式被称为 Applicative Order。但实际上至少还有另一种 evaluation 的方式，就是推迟 sub-expression 的 evaluation 过程，先把 sub-expression 代入到 application expression，直到某个阶段必须要获取 sub-expression 的值时才去 evaluate sub-expression。总结如下：

* Applicative Order: 在 evaluate expression 前先 evaluate sub-expression
* Normal Order: 只在需要的时候 evaluate sub-expression

#### 举例

```scheme
> ; applicative order
> (define (foo x)
    (write-line "inside foo")
    (+ x x))

> (foo (begin (write-line "eval arg") 222))
eval arg
inside foo
=> 444

> ; normal order
> (define (foo x)
    (write-line "inside foo")
    (+ x x))
> (foo (begin (write-line "eval arg") 222))

; => (begin (write-line "inside foo")
;      (+ (begin (write-line "eval arg") 222)
;         (begin (write-line "eval arg") 222)))
inside foo
eval arg
eval arg
=> 444
```

可以看见，如果用 Normal Order，sub-expression 在最后要执行 &lt;procedure +&gt; 的时候才被 evaluate，因此 "inside foo" 比 "eval arg" 更早被打印出来。但由于 sub-expression 被延迟 evaluate，它被执行了两次，显然这对 Normal Order 的 evaluation 方式来说是劣势。自然的，有两个疑问需要解决：

* Normal Order 的好处是什么？
* 怎么解决 Normal Order 带来的多次 evaluation？
* 如何改变我们的 Scheme 解释器去实现 Normal Order Evaluation ？（接下来先解决）

### Normal Order Scheme Interpreter

为了让 Scheme 解释器执行 Normal Order Evaluation，我们需要做出以下几个改动：

1. 改变 apply 逻辑
   * 记录 env: 因为 arguments \(sub-expressions\) 的 evaluation 被推迟，那么它们的 evaluation environment 就需要被记录
   * 记录 delayed arguments 的数据结构
2. 支持 actual value 和 delayed value
3. if 表达式

##### 改变 apply 逻辑

```scheme
(define (l-apply procedure arguments env)          ; env
  (cond (primitive-procedure? procedure)
        (apply-primitive-procedure
          procedure
          (list-of-arg-values arguments env)))     ; list-of-arg-values
        ((compound-procedure? procedure)
         (l-eval-sequence
           (procedure-body procedure)
           (extend-environment
             (procedure-parameters procedure)
             (list-of-delayed-args arguments env)  ; list-of-delayed-args
             (procedure-environment procedure))))
         (else (error "Unkown proc" procedure))))

; 对应使用 apply 的地方也要改动
(define (l-eval exp env)
  (cond ((self-evaluating? exp) exp)
        ; ...
        ((application? exp
         (l-apply (actual-value (operator exp) env)
                  (operands exp)
                  env))
        (else (error "Unkown expression" exp))))
```

主要有三处变化：

* env: 上文提过，为了推迟 evaluation ，需要记录相应的环境
* list-of-arg-values: 当执行 primitive-procedure 的时候，强制 evaluate 所有 delayed arguments
* list-of-delayed-args: 将 arguments 和 env 打包起来，在之后随时可以强制 evaluate

首先我们看一下 list-of-arg-values 和 list-of-delayed-args 的实现

```scheme
(define (actual-value exp env)
  (force-it (l-eval exp env)))

(define (list-of-arg-values exps env)
  (if (no-operands? exps) '()
    (cons (actual-value (first-operand exps) env)
          (list-of-arg-values (rest-operands exps)
                              en))))

(define (list-of-delayed-args exps env)
  (if (no-operands? exp) '()
    (cons (delay-it (first-operand exps) env)
          (list-of-delayed-args (rest-operands exps)
                                env))))
```

##### thunk - promise to do an evaluation

以下几个词都与 thunk 同义：

* delayed value
* promise to do an evaluation
* promise to return a value when needed, i.e. forced

延用 data driven programming 的风格，我们可以利用 tagged list 来表示 thunk:

```scheme
(define (delay-it exp env) (list 'thunk exp env))
(define (thunk? obj) (tagged-list? obj 'thunk))
(define (thunk-exp thunk) (cadr thunk))
(define (thunk-env thunk) (caddr thunk))

(define (force-it obj)
  (cond ((thunk? obj)
         (actual-value (thunk-exp obj)
                       (thunk-env obj)))
        (else obj)))
```

##### If expression

一些特殊表达式也需要修改，如 if 的 condition 必须要有 actual-value 才可以判断下一步是 consequence 还是 alternative:

```scheme
(define (l-eval-if exp env)
  (if (true? (actual-value) (if-predicate exp) env))
      (l-eval (if-consequent exp) env)
      (l-eval (if-alternative exp) env)))
```

以上，我们完成了对 Scheme 解释器的 evaluator 改造，现在它已经从 Applicative Order 变成了 Normal Order。

### Memo-izing evaluation

在 Applicative Order & Normal Order 一节中，我们观察到，Normal Order 由于推迟 sub-expression 的 evaluation，而把 sub-expression 替换到 application expression 的相应位置，同时带来了重复计算 \(re-evaluation\) 的情况，本节我们介绍如何利用 Memo-izing Thunks 来避免重复计算。

Memo-izing Thunk 的原理再简单不过，计算一次以后就把结果记下来，下次用到的时候再拿出来即可。

具体示意图如下：

（图1）

需要注意的是，为了使用 thunk 的所有地方都能立即看到 thunk 被 evaluate 过，这里使用 mutation 把 thunk 编程 evaluated-thunk:

```scheme
(define (evaluated-thunk? obj)
  (tagged-list? obj 'evaluated-thunk))
(define (thunk-value evaluated-thunk)
  (cadr evaluated-thunk))

(define (force-it obj)
  (cond ((thunk? obj)
         (let ((result (actual-value (thunk-exp obj)
                                     (thunk-env obj)))))
            (set-car! obj 'evaluated-thunk)
            (set-car! (cdr obj) result)
            (set-cdr! (cdr obj) '())
            result))
        ((evaluated-thunk? obj) (thunk-value obj))
        (else obj))
```

### Normal Order And Language Design \(pros and cons\)

Language Design Choice 和人生中的每一个选择一样，都是 trade-off，选择 Normal Order 的 trade-off 如下：

* pros：只在必要的时候计算
* cons:
  1. 可能造成重复计算
  2. 不确定计算发生的时间

Memo-izing evaluation 似乎可以解决 cons1，但有时候我们希望 expression 每次都被重新计算。因此解决 Normal Order 的 cons 的本质在于让程序员来决定什么时候 evaluate expression，举例如下：

```scheme
(lambda (a (b lazy) c (d lazy-memo)) ;...)
```

* a、c: applicative order
* b: 每次需要的时候都重新计算
* d: 只计算一次就记住结果

```scheme
(define (first-variable var-decls) (car var-decls))
(define (rest-variables var-decls) (cdr var-decls))
(define declaration? pair?)

(define (parameter-name var-decl)
  (if (pair? var-decl) (car var-decl) var-decl))

(define (lazy? var-decl)
  (and (pair? var-decl)
       (eq? 'lazy (cadr var-decl))))

(define (memo? var-decl)
  (and (pair? var-decl)
       (eq? 'lazy-memo (cadr var-decl))))

; 需要一直支持 value, lazy, lazy-memo 的 delay-it
(define (delay-it decl exp env)
  (cond ((not (declaration? decl))
         (l-eval exp env))
        ((lazy? decl)
         (list 'thunk exp env))
        ((memo? decl)
         (list 'thunk-memo exp env))
        (else (error "unkown declaration:" decl))))

; 以及新的 force-it
(define (force-it obj)
  (cond ((thunk? obj)                     ; eval, but don't remember it
         (actual-value (thunk-exp obj)
                       (thunk-env obj)))
        ((memoized-thunk? obj)            ; eval and remember
         (let ((result (actual-value (thunk-exp obj)
                                     (thunk-env obj))))
           (set-car! obj 'evaluated-thunk)
           (set-car! (cdr obj) result)
           (set-cdr! (cdr obj) '())
           result))
        ((evaluated-thunk? obj) (thunk-evalue obj))
        (else obj)))
        
; 修改 l-apply
(define (l-apply procedure arguments env)          
  (cond (primitive-procedure? procedure)
        (apply-primitive-procedure
          procedure
          (list-of-arg-values arguments env)))     
        ((compound-procedure? procedure)
         (l-eval-sequence
           (procedure-body procedure)
           (let ((params (procedure-parameters procedure)))
             (extend-environment
               (map parameter-name params)                  ; 取出参数名
               (list-of-delayed-args params arguments env)  ; 把参数的设置 (actual value、lazy、lazy-memo) 传递下去
               (procedure-environment procedure)))))
         (else (error "Unkown proc" procedure))))

(define (list-of-delayed-args var-decls exps env)
  (if (no-operations? exps)
      '()
      (cons (delay-it (first-variable var-decls)
                      (first-operand exps)
                      env)
            (list-of-delayed-args
              (rest-variables var-decls)
              (rest-operands exps)
              env))))
```

现在我们的 Scheme Evaluator 已经支持自由定义参数的计算方式：actual value、lazy 以及 lazy-memo。

