# 第十八课 - Scheme 解释器中的 Eval-Apply loop

第十七课构建了一个简单 Scheme 的 Evaluator，本节将深入讨论 Scheme 真正的 Evaluator。

### Scheme 解释器的五个组件

#### 组件1：eval/apply core

eval 和 apply 是 Scheme 解释器中 Evaluator 的核心组件，它们一同定义 Scheme 语言的 semantics。

（图1）

如上图所示，Scheme 解释器接收合法的表达式 \(exp\) 以及表达式所处的环境 \(env\) 后，会通过 eval  procedure 来在这个环境中 evaluate 这个表达式。复杂表达式通常会包含另一个 application 表达式，这个表达式会被 eval procedure 进一步处理成更简单的表达式 \(exp\) 及其由新生成的 frame 以及之前的环境一同组成的新环境，然后再次被 eval procedure 进行处理。这个过程周而复始，直到表达式变成 primitive procedure 为止。

总结一下 eval 和 apply 的职责：

##### eval

根据表达式的类型来把表达式分发给对应的 procedure ，以下就是 eval 的具体代码实现

```scheme
(define (m-eval exp env)
  (cond ((self-evaluating? exp) exp)
        ((variable? exp) (look-up-variable-value exp env))
        ((quoted? exp) (text-of-quotation exp))
        ((assignment? exp) (eval-assignment exp env))
        ((definition? exp) (eval-definition exp env))
        ((if? exp) (eval-if exp env))
        ((lambda? exp)
         (make-procedure (lambda-parameters exp) (lambda-body exp) env))
        ((begin? exp) (eval-sequence (begin-actions exp) env))
        ((cond? exp) (m-eval (cond-if exp) env))
        ((application? exp)
         (m-apply (m-eval (operator exp) env)
                (list-of-values (operands exp) env)))
        (else (error "Unknown expression type -- EVAL" exp))))
```

这里需要十分注意 eval 中各个类型表达式的 dispatch 顺序：

1. 检查普通表达式，如self-evaluating 、variable 以及 quote 等等这些简单的表达式
2. 检查特殊形式，如 assignment、define 这些有副作用的表达式，if、cond、begin 这些流程控制表达式，以及 lambda 表达式
3. application 表达式，它可以是编程者定义的任意 procedure，接收自定义的参数，执行自定义的过程。通常，如果一个表达式是复合表达式且不属于任意已知的特殊表达式，它就会被认为是一个 application 表达式。

##### **apply**

会首先 eval 表达式中的参数，即表达式的 operands，然后再将 eval 后的参数传递给表达式中的 procedure，即表达式的 operator，最后 eval 这个 procedure，这个 procedure 可以是 primitive procedures 也可以是用户自定义的一般 procedures。在计算模型引入 mutation 之后，Scheme 的 procedure 通常需要支持执行多个表达式，并以最后一个表达式的返回值来代表整体表达式的返回值，因此 m-apply 需要支持执行 body 中含有多个表达式的 procedure:

```scheme
(define (m-apply procedure arguments)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure procedure arguments))
        ((compound-procedure? procedure)
         (eval-sequence
          (procedure-body procedure)
          (extend-environment (procedure-parameters procedure)
                              arguments
                              (procedure-environment procedure))))
        (else (error "Unknown procedure type -- APPLY" procedure))))

(define (eval-sequence exps env)
  (cond ((last-exp? exps) (m-eval (first-exp exps) env))
        (else (m-eval (first-exp exps) env)
              (eval-sequence (rest-exps exps) env))))
```

现在我们已经可以体会到 eval 和 apply 之间的互相踢皮球的合作模式，这有点像 EM 算法里的 E step 和 M step （没见过请忽略这句）...

但我们还没体会到 "eval 和 apply 定义了 Scheme 的 semantics" 。我们先用大白话解释一下 semantics 的意思。这里 semantics 指的是编程语言背后的意义，这个意义与具体的语法无关。比如要表达 “我爱你”，不同的语言有不同的语法，但被后的意义都是一个主体对另一个主体的感情。接下来我们将通过下一个组件的介绍来体会这句话。

#### 组件2：syntax procedures

syntax procedures 决定语言的语法，即任意表达式的合法性。在 Scheme 解释器中，首先我们需要一些常用 procedure 来检测表达式的类型：

```scheme
(define (self-evaluating? exp)
  (or (number? exp)
      (string? exp)
      (boolean? exp)))

(define (tagged-list? exp tag)
  (and (pair? exp) (eq? (car exp) tag)))

(define (quoted? exp) (tagged-list? exp 'quote))
(define (text-of-quotation exp) (cadr exp))

(define (variable? exp) (symbol? exp))
(define (assignment? exp) (tagged-list? exp 'set!))
(define (assignment-variable exp) (cadr exp))
(define (assignment-value exp) (caddr exp))

; ... definition
(define (definition? exp) (tagged-list? exp 'define))

(define (lambda? exp) (tagged-list? exp 'lambda))
(define (lambda-parameters lambda-exp) (cadr lambda-exp))
(define (lambda-body lambda-exp) (cddr lambda-exp))
(define (make-lambda params body) (cons 'lambda (cons params body)))

;... if, cond, begin, ..
(define (application? exp) (pair? exp))
(define (operators app) (car app))
(define (operands app) (cdr app))
```

完整代码可见参考文献。这些 syntax procedures 定义了 Scheme 的各种合法表达式的语法。仔细思考不难发现，在解释器中，Scheme 语言的 semantics 和 syntax 是互相解耦的，只要我们改变了这些 syntax procedures，就可以改变 Scheme 的 syntax，但对它的 semantics 却没有影响。接下来举几个例子：

##### 例1：\(CALL &lt;proc&gt; ARGS &lt;arg1&gt; &lt;arg2&gt; ...\)

假设 Scheme 的设计者希望用更具体的表达式来执行一个 procedure，如标题所示，我们需要对原先的解释器代码作哪些改动呢？

```scheme
(define (application? exp) (tagged-list? 'CALL))
(define (operator app) (cadr app))
(define (operands app) (cdddr app))
```

搞定！我们只修改了 syntax，而语言的 semantics 不受影响。

##### 例2：Syntactic Sugar - let -&gt; lambda

我们还可以添加语法糖，比如将 let binding 转化成 lambda：

```scheme
(let ((<name> <val1>)
       (<name> <val2>))
   <body>)
 ; =>
 ((lambda (<name1> <name2>) <body>)
          (<val1> <val2>))
```

我们又需要改变什么呢？

* 在 m-eval 的 cond 中增加处理 let 表达式的情况，并把处理的工作 dispatch 给 let-&gt;combination procedure
* 实现 let 表达式到 lambda 表达式的转化，即 let-&gt;combination

```scheme
(define (m-eval exp env)
  (cond ((...))
        ...
        ((let? exp)
         (m-eval (let->combination exp) env))
        ((application? exp)
         ...
        (else ...)))


(define (let? exp) (tagged-list? exp 'let))
(define (let-bound-variables let-exp)
  (map car (cadr let-exp)))
(define (let-values let-exp)
  (map cadr (cadr let-exp)))
(define (let-body let-exp)
  (sequence-> exp (cddr let-exp)))

(define (let->combination let-exp)
  (let ((names (let-bound-variables let-exp))
        (values (let-values let-exp))
        (body (let-body let-exp)))
    (cons (list 'lambda names body) values)))
```

我们知道在 Scheme 中，除了原始数据类型，剩下的东西都是 list，那么从 let 表达式转化成 lambda 表达式的过程，实际上就是一个 list 到另一个 list 的转化过程，如下面这个表达式：

```scheme
(let ((x 23)
      (y 15))
  (do-something x y))
```

经过 let-combination 转化后就会变成：

```scheme
((lambda (x y)
   (do-somthing x y))
 23 15)
```

###### let 表达式示意图

\(图2\)

###### lambda 表达式示意图

（图3）

##### 例3：Syntactic Sugar - named procedures

在当前环境下，如果我们要在环境中定义 procedure，即 named procedure，我们必须这么写：

```scheme
(define m-sum (lambda (x y) (+ x y))
```

如果我们想更简单一点：

```scheme
(define (m-sum x y) (+ x y))
```

那世界就更美好一点。怎么做？

首先看一下处理 definition 的 procedure:

```scheme
(define (eval-definition exp env)
  (define-variable! (definition-variable exp)
                    (m-eval (definition-value exp) env)
                    env))
(define (definition-variable exp) (cadr exp))
(define (definition-value exp) (caddr exp))
```

与前面的例子相似，我们只需要修改 syntax 而不需要改动 semantics，这里我们希望 definition-variable 和 definition-value 同时兼容两种形式：

```scheme
(define (definition-variable exp)
  (if (symbol? (cadr exp)) (cadr exp) (caadr exp)))
(define (define-value exp)
  (if (symbol? (cadr exp))
      (caddr exp)
      (make-lambda (cdadr exp) (cddr exp))))
```

搞定！现在相信你对 “eval 和 apply 定义了 Scheme 的 semantics” 这句话能够有所体会。当我们需要修改语法时，只需要增删改处理不同类别表达式的 procedures。

#### 组件3：environment manipulation

之前我们提到环境模型的实现时，假设背后已经存在一个 table ADT，现在来看一下具体的实现。

首先重新整理一遍对环境的需求：

* 支持往环境中添加新的 binding
* 环境拥有外环境指针，能根据外环境指针找到它的外环境 \(enclosing environment\)，以支持环境链。

我们可以用 list 来完成这些需求。

抽象地看，我们心中的环境模型如下图所示：

（图4）

具体地看，我们心中的环境模型如下图所示：

（图5）

extend-environment 时，抽象地看，我们心中的环境模型如下图所示：

（图6）

extend-environment 时，具体地看，我们心中的环境模型如下图所示：

（图7）

对应的代码如下：

```scheme
(define (make-frame variables values) (cons variables values))
(define (add-binding-to-frame! var val frame)
  (set-car! frame (cons var (car frame)))
  (set-cdr! frame (cons val (cdr frame))))
(define (extend-environment vars vals base-env)
  (if (= (length vars) (length vals))
      (cons (make-frame vars vals) base-env)
      (if (< (length vars) (length vals))
          (error "Too many args supplied" vars vals)
          (error "Too few args supplied" vars vals))))
```

使用时，需要从环境链中搜索出对应的 binding:

```scheme
; helper
(define (enclosing-environment env) (cdr env))
(define (first-frame env) (car env))
(define the-empty-environment '())
(define (frame-variables frame) (car frame))
(define (frame-values frame) (cdr frame))

(define (lookup-variable-value var env)
  (define (env-loop env)
    (define (scan vars vals)
      (cond ((null? vars) (env-loop (enclosing-environment env)))
            ((eq? var (car vars)) (car vals))
            (else (scan (cdr vars) (cdr vals)))))
    (if (eq? env the-empty-environment)
        (error "Unbound variable -- LOOKUP" var)
        (let ((frame (first-frame env)))
          (scan (frame-variables frame) (frame-values frame)))))
  (env-loop env))
```



