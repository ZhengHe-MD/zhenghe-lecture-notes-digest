# 第十八课 - Scheme 解释器中的 Eval-Apply loop

第十七课构建了一个简单 Scheme 的 Evaluator，本节将深入讨论 Scheme 真正的 Evaluator。

### Scheme 解释器的五个组件

#### 组件1：eval/apply core

eval 和 apply 是 Scheme 解释器中 Evaluator 的核心组件，它们一同定义 Scheme 语言的 semantics。

![](/assets/Screen Shot 2018-04-09 at 10.09.29 PM.jpg)

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

![](/assets/Screen Shot 2018-04-09 at 10.09.56 PM.jpg)

###### lambda 表达式示意图

![](/assets/Screen Shot 2018-04-09 at 10.10.26 PM.jpg)

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

![](/assets/Screen Shot 2018-04-09 at 10.10.50 PM.jpg)

具体地看，我们心中的环境模型如下图所示：

![](/assets/Screen Shot 2018-04-09 at 10.11.12 PM.jpg)

extend-environment 时，抽象地看，我们心中的环境模型如下图所示：

![](/assets/Screen Shot 2018-04-09 at 10.11.34 PM.jpg)

extend-environment 时，具体地看，我们心中的环境模型如下图所示：

![](/assets/Screen Shot 2018-04-09 at 10.12.09 PM.jpg)

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

#### 组件4：primitives and initial env

Global Environment 以及其中的 primitive procedures 是功能完整的 Scheme 解释器不可缺少的一部分，否则我们需要重复制造许多轮子：

```scheme
(define primitive-procedures
  (list (list 'car car)
        (list 'cdr cdr)
        (list 'cons cons)
        (list 'null? null?)
        (list '+ +)
        (list '> >)
        (list '= =)
        (list '* *)
        ; ... more primitives
  ))

(define (setup-environment)
  (let ((initial-env (extend-environment 
                      (primitive-procedure-names)
                      (primitive-procedure-objects)
                      the-empty-environment)))
    (define-variable! 'true #t initial-env)
    (define-variable! 'false #f initial-env)
    initial-env))
(define the-global-environment (setup-environment))
```

#### 组件5：read-eval-print loop

和解释器的 evaluator 交互，需要 read-eval-print loop \(REPL\) 组件，它读取用户输入、eval 表达式、打印结果然后再次等待用户输入，不断循环。

```scheme
(define (driver-loop)
  (prompt-for-input input-prompt)
  (let ((input (read))
    (let ((output (m-eval input the-global-env)))
      (announce-output output-prompt)
      (user-print output)))
  (driver-loop))
```

### Scoping

不论是何种语言都有 Scoping 的概念，以 Scheme 为例：当我们 evaluate procedures 的时候，在 procedure body 中会遇到两种不同的 symbol:

* bound parameters: 在 procedure 的参数中定义过的 symbol
* free variables: 在 procedure 的参数中未定义过的 symbol

我们如何找到 free variables 对应的值就是所谓的 scoping。

#### Lexical Scoping

Lexical Scoping 指的就是在 procedure 被定义时的环境中寻找 free variables 的 bindings。具体可以看一下 make-procedure 的实现：

```scheme
(define (make-procedure parameters body env)
  (list 'procedure parameters body env))
```

make-procedure 在创建 procedure 的过程中把当前环境也保存在其中，它将被用于寻找 free variables 的 bindings。在 Lexical binding 的情形中：

```scheme
> (define (foo x y)
    (lambda (z) (+ x y z)))
> (define bar (foo 1 2))
> (bar 3)
6
```

其中 foo 的内环境是 **x = 1, y = 2**，外环境是 **GE**；bar 的内环境是 **z = 3**，外环境是 **foo 的内环境**。因此在 eval \(bar 3\) 的时候，会在 bar 的内环境中找到 z，bar 的外环境 （即 foo 的内环境）中找到 free variables - x, y 对应的值。

#### Dynamic Scoping

Dynamic scoping 指的就是在 procedure 被调用时的环境中寻找 free variables 的 bindings。在 dynamic scoping 的情形中：

```scheme
> (define (pooh x)
    (bear 20))
> (define x 3)
> (define (bear y)
    (+ x y))
> (pooh 9)
29
> (bear 20)
23
```

在 eval 表达式 \(pooh 9\) 和表达式 \(bear 20\) 的过程中，bear procedure 的调用环境不同，因此得到的 free variable x 的值不同，eval 的结果也就不同。如果你熟悉 ECMAScript，阅读完本节后，ECMAScript 中的 this binding 对你来说应该很容易理解。

#### Dynamic Scoping Scheme

显然，目前构建的 Scheme 解释器采用的是 Lexical Scoping，如果我们想造一个 Dynamic Scoping 的 Scheme，需要怎么做呢？本节课一直强调的一句话是 "eval 和 apply 定义了 Scheme 的 semantics"，显然 Scoping 属于 semantics，因此要改变 Scoping 当然应当修改 eval 和 apply。

首先 make-procedure 不需要保留环境

```scheme
(define (make-procedure parameters body)
  (list 'procedure parameters body))

(define (m-eval exp env)
  (cond
    ;...
    ((lambda? exp)
      (make-procedure (lambda-parameters exp)
        (lambda-body exp))) ; 去除 env
    ;...
    ((application? exp)
     (d-apply (m-eval (operator exp) env)
              (list-of-values (operands exp) env)
              env))         ; 新增 env
    (else (error "Unkown expression -- M-EVAL" exp))))
```

然后再修改 d-apply ，在 extend-environment 的时候使用 calling environment 而不是 procedure environment

```scheme
(define (d-apply procedure arguments calling-env)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure procedure arguments))
        ((compound-procedure? procedure)
         (eval-sequence
           (procedure-body procedure)
           arguments
           calling-env)))
        (else (error "Unkown procedure" procedure))))
```

搞定！这里我们再次体会到了 semantics 与 syntax 的分离的好处，改变 semantics 中的 scoping 的同时 syntax 不受丝毫影响。

### 小结

* eval 和 apply 定义了 Scheme 的 semantics

* syntax procedures 定义了 Scheme 的 syntax

* semantics 和 syntax 的隔离使得二者可以单独改变而不影响对方

#### 参考

* [Youtube: SICP-2004-Lecture-18](https://www.youtube.com/watch?v=B2SIMf1gPHc&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=18&t=0s)
* [MIT6.006-SICP-2005-Lecture-notes-18](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture20webhan.pdf)
* [MIT6.006-SICP-2005-Lecture-codes](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture20evalco.pdf)



