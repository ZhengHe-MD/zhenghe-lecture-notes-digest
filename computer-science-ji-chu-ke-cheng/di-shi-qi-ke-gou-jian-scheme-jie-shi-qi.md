# 第十七课 - 构建 Scheme 解释器

我们用编程语言写的每一句表达式，都有它的含义，将表达式推导转化成它的实际含义的过程，被称为 evaluation。而实现这个推导转化过程的程序，就是解释器（interpreter）。

### 解释器的基本构造

解释器通常由 Lexical Analyzer、Parser、Evaluator & Environment、Printer 四个部分组成，它们的关系如下图所示：

![](/assets/Screen Shot 2018-04-07 at 4.23.21 PM.jpg)

解释器的输入一般为一句合法的表达式的字符串，如 "\(average 4 \(+ 5 5\)\)"，该字符串将依次经过这四个组件处理

* Lexical Analyzer: 将字符串转化成有意义的单词，称作 token
* Parser：将各单词组合成便于推导转化的树状结构 （tree structure）
* Evaluator & Environment：结合环境中的其它变量，将输入的树状结构推导转化成最终的结果值
* Printer：将结果值转化成字符串输出到屏幕上

本节课的重心将放在 Evaluator 这部分，由于是写一个 Scheme 的解释器，因此 Lexical Analyzer 和 Parser 这两部分可以直接利用 Scheme 本身的相关组件。由于要构建的语言和 Scheme 本身比较像，为了与 Scheme 本身区分，我们的新语言名称叫作 Scheme\*，与 Scheme 内置的方法功能重复的方法也都类似地会使用 \* 以示区分。

### 构造 Scheme\* 解释器

#### Arithmetic calculator

首先，我们尝试 evaluate 以下的加法表达式：

```scheme
(plus* 24 (plus* 5 6))
```

主要代码如下所示，核心过程为 eval

```scheme
(define (tag-check e sym)
  (and (pair? e) (eq? (car e) sym)))
(define (sum? e) (tag-check e 'plus*))

(define (eval exp)
  (cond
    ((number? exp) exp)
    ((sum? exp) (eval-sum exp))
    (else
      (error "unknown expression" exp))))

(define (eval-sum exp)
  (+ (eval (cadr exp)) (eval (caddr exp))))

(eval '(plus* 24 (plus* 5 6)))
```

在表达式 **'\(plus\* 24 \(plus\* 5 6\)\)** 被读入 Scheme 解释器时，它就已经被 Scheme 的 Lexical Analyzer 和 Parser 处理成了如下树状结构：

![](/assets/Screen Shot 2018-04-07 at 4.23.54 PM.jpg)

eval 过程通过判断表达式的第一个 token 的类型，来决定要对其进行什么样的操作 --- 如果是 number，就直接返回对应数值，如果是 plus\*，则将表达式交给 eval-sum 去递归推导转化成最终结果。

上文代码利用数据驱动及防御式的编程方式组织 eval 过程，同时利用递归的方式将复杂表达式一层一层剥开，直到最简单的表达式，然后再将结果一层一层组合，最终推导得到计算表达式的结果。这就是一个最简单的 evaluator。

#### Names

接下来，我们要在 sheme\* 中加入 names，也就是 define\* 表达式：

```scheme
(define* x* (plus* 4 5))
(plus* x* 2)
```

首先，我们需要一种 Table ADT，可以让我们存放和获取每个 name 与 value 的 binding 关系。

```
; make-table             void -> table
; table-get              table, symbol -> (binding | null)
; table-put!             table, symbol, anytype -> undef
; binding-value          binding -> anytype
```

假设这个 Table ADT 存在，scheme\* 解释器就拥有支持 name binding 的能力：

```scheme
(define (define? exp) (tag-check exp 'define*))

(define (eval exp)
  (cond
    ((number? exp) exp)
    ((sum? exp) (eval-sum exp))
    ((symbol? exp) (lookup exp))
    ((define? exp) (eval-define exp))
    (else
      (error "unknown expression" exp))))

(define environment (make-table))

(define (lookup name)
  (let ((binding (table-get environment name)))
    (if (null? binding)
        (error "unbound variable: " name)
        (binding-value binding))))

(define (eval-define exp)
  (let ((name (cadr exp))
        (defined-to-be (caddr exp)))
    (table-put! environment name (eval define-to-be)) 'undefined))
```

define\* 表达式与之前的 plus\* 表达式不同。plus\* 表达式会 evaluate 它的两个参数，我们称这种表达式为一般表达式；define\* 表达式只会 evaluate 第二个参数，把第一个参数当作 symbol，我们称不 evaluate 所有输入参数的表达式为特殊表达式 \(special forms\)。

#### Conditionals and If

scheme\* 少不了 predicates 和条件语句 if

```scheme
(define (greater? exp) (tag-check exp 'greater*))
(define (if? exp) (tag-check exp 'if*))

(define (eval exp)
  (cond
    ((number? exp) exp)
    ((sum? exp) (eval-sum exp))
    ((symbol? exp) (lookup exp))
    ((define? exp) (eval-define exp))
    (else
      (error "unknown expression" exp))))

(define (eval-greater exp)
  (> (eval (cadr exp)) (eval (caddr exp))))

(define (eval-if exp)
  (let ((predicate   (cadr exp))
        (consequent  (caddr exp))
        (alternative (cadddr exp)))
    (let ((test (eval predicate)))
      (cond
        ((eq? test #t) (eval consequent))
        ((eq? test #f) (eval alternative))
        (else          (error "predicate not a conditional: " predicate))))))
```

#### Store operators in the environment

既然有了 greater\*, sum\* 肯定少不了 lessThan\*、equal\*、difference\*、mod\* 等各种操作，如果逐一添加就会显得代码十分冗余。仔细观察可以发现，这些操作都有一个特点：先 evaluate 参数，然后对这些参数执行相应操作。这种模式就是上文提到的一般表达式，也称为 application。

为了达到目的，我们需要把这些操作存到环境中：

```scheme
(define scheme-apply apply) ; 保存 scheme 内置的 apply

(define (apply operator operands)
  (if (primitive? operator)
      (scheme-apply (get-scheme-procedure operator) operands)
      (error "operator not a procedure: " operator)))

(define prim-tag 'primitive)
(define (make-primitive scheme-proc) (list prim-tag scheme-proc))
(define (primitive? e)               (tag-check e prim-tag))
(define (get-scheme-procedure prim)  (cadr prim))

(define environment (make-table))
(table-put! environment 'plus*     (make-primitive +))
(table-put! environment 'greater*  (make-primitive >))
(table-put! environment 'true* #t)
```

然后将一般表达式统一起来：

```scheme
(define (eval exp)
  (cond
    ((number? exp)       exp)
    ((symbol? exp)       (lookup exp))
    ((define? exp)       (eval-define exp))
    ((if? exp)           (eval-if exp))
    ((application? exp)  (apply (eval (car exp))
                                (map eval (cdr exp))))
    (else
      (error "unknown expression " exp))))
```

这是 scheme\* 解释器已经初具雏形，递归地 evaluate 表达式，特殊表达式在前，一般表达式在后。

#### Environment as explicit parameter

虽然已经有了 global environment，但很多情况下我们仍然需要 local environment 的支持。拥有 local environment 意味着 local environment 需要成为过程的输入参数，这样就可以使得同样的过程在不同的 local environment 中执行：

```scheme
(define (eval exp env)
  (cond
    ((number? exp)       exp)
    ((symbol? exp)       (lookup exp env))
    ((define? exp)       (eval-define exp env))
    ((if? exp)           (eval-if exp env))
    ((application? exp)  (apply (eval (car exp) env)
                                 (map (lambda (e) (eval e env))
                                      (cdr exp))))
    (else
      (error "unknown expression " exp))))

(define (lookup name env)
  (let ((binding (table-get env name)))
    (if (null? binding)
        (error "unbound variable: " name)
        (binding-value binding))))

(define (eval-define exp env)
  (let ((name (cadr exp))
        (defined-to-be (caddr exp)))
    (table-put! env name (eval defined-to-be env))
    'undefined))

(define (eval-if exp env)
  (let ((predicate    (cadr exp))
        (consequent   (caddr exp))
        (alternative  (cadddr exp)))
    (table-put! env name (eval defined-to-be env))
    'undefined))
```

这里比较费解的地方在于 \(application? exp\) 后的内容。由于一个 application 的所有参数也需要在同一个 environment 中 evaluate，因此这里利用了 lambda 和 map 依次在同一个 environment 中 evaluate 每个参数。

#### Add new procedures

在 Scheme 中，我们可以用 lambda 自定义 procedures，lambda 表达式会返回一个在 environment model 中被称为 double bubble 的 compound procedure。本节，我们也将在 Scheme\* 解释器中支持 lambda\* 特殊表达式。

lambda\* 表达式有三个重要成部分，参数、函数体以及环境，因此在 eval lambda\* 的时候，需要在返回的数据结构中保存这三部分信息。在这里我们称这个数据结构为 compound，也就是上文到的 double bubble 或 compound procedure：

```scheme
(define (lambda? e) (tag-check e 'lambda*))

(define (eval exp env)
  (cond
    ((number? exp)       exp)
    ((symbol? exp)       (lookup exp env))
    ((define? exp)       (eval-define exp env))
    ((if? exp)           (eval-if exp env))
    ((lambda? exp)       (eval-lambda exp env))
    ((application? exp)  (apply (eval (car exp) env)
                                 (map (lambda (e) (eval e env))
                                      (cdr exp))))
    (else
      (error "unknown expression " exp))))

(define (eval-lambda exp env)
  (let ((args (cadr exp))
        (body (caddr exp)))
    (make-compound args body env)))


;; ADT that implements the "double bubble"
;; 这里 compound 由 parameters、body、env 三部分构成，ADT 同时提供三者的 selectors
(define compound-tag 'compound)
(define (make-compound parameters body env)
    (list compound-tag parameters body env))
(define (compound? exp) (tag-check exp compound-tag))
(define (parameters compound) (cadr compound))
(define (body compound) (caddr compound))
(define (env compound) (caddr compound))
```

在 eval lambda\* 表达式时，我们并没有执行实际操作，而只是老老实实地把自定义 procedures 的元素保存在 list 结构中，然后返回这个 list。当然，这里是否使用 list 并不重要，这对解释器的使用者透明。

那我们如何 eval compound 呢？实际上就是实现在环境模型一节中介绍的过程：

1. 创建一个新的 Frame，设为 A
2. 将 A 的外环境指针指向 P 的外环境指针所指的环境，此时 A 及其外环境一起构成新的环境，设为 E
3. 在 A 中，将 P 的参数与它的值绑定起来
4. 在 E 中执行 P 的程序体

```scheme
(define (apply operator operands)
  (cond ((primitive? operator)
         (scheme-apply (get-scheme-procedure operator)
                       operands))
        ((compound? operator)
         (eval (body operator)
               (extend-env-with-new-frame
                 (parameters operator)
                 operands
                 (env operator))))
       (else
         (error "operator not a procedure: " operator))))

(define (extend-env-with-new-frame names values env)
  (let ((new-frame (make-table)))
    (make-bindings! names values new-frame)
    (cons new-frame env))) ; 注意：这里 new-frame 在 env 之前，体现 lookup 顺序

(define (make-bindings names values table)
  (for-each
    (lambda (name value) (table-put! table name value))
    names values))

; the initial global environment
(define GE
  (extend-env-with-new-frame
    (list 'true* 'plus* 'greater*)
    (list #t (make-primitive +) (make-primitive >))
    nil))

; lookup searches the list of frames for the first match
(define (lookup name env)
  (if (null? env)
      (error "unbound variable: " name)
      (let ((binding (table-get (car env) name)))
        (if (null? binding)
            (lookup name (cdr env))
            (binding-value binding)))))

; define changes the first frame in the environment
(define (eval-define exp env)
  (let ((name           (cadr exp))
        (defined-to-be  (caddr exp)))
    (table-put! (car env) name (eval defined-to-be env))
    'undefined)
```

实现上，所谓的外环境指针并不真实存在，而是通过 const 将内外环境合成一个 list，利用 list 结构的先后顺序来保证 lookup 从内环境到外环境的顺序一致。

#### 举例：

```scheme
(eval '(define* twice*
  (lambda* (x*) (plus* x* x*))) GE)
```

执行完以上 define\* 语句后，GE 变为：

![](/assets/Screen Shot 2018-04-07 at 4.24.05 PM.jpg)

这时候，twice\* 进入 GE 后，就可以调用 twice procedure

```scheme
(eval '(twice* 4) GE)
```

首先 **'\(twice\* 4\)** 是 application，因此得到

```scheme
(apply (eval 'twice* GE)
  (map (lambda (e) (eval e GE)) '(4)))
```

map 表达式在 GE 中 evaluate twice\* 的参数 --- 4：

```scheme
(apply (eval 'twice* GE) '(4))
```

GE 中可以找 'twice\*

```scheme
(apply (list 'compound '(x*) '(plus* x* x*) GE) '(4))
```

apply 发现后面的表达式是 compound procedure

```scheme
(eval '(plus* x* x*)
  (extend-env-with-new-frame '(x*) '(4) GE))
```

仔细阅读 GE 的代码，实际上 environment 并不是 table，而是 list&lt;table&gt;，extend-env-with-new-frame 实际上就是在当前 env 的基础之上，在 list 的前面 const 一个 table，代表 new frame。假设 table ADT 已经存在：

```scheme
(eval '(plus* x* x*)
  ((list 'table 'new-frame) (list 'table 'GE)))
> 8
```

这里需要注意，如果 \(eval  '\(twice\* 4\) 时的 env 并不是 GE，则以上过程将变成

```scheme
(eval '(twice* 4) AE) ; another environment
(apply (eval 'twice* AE)
  (map (lambda (e) (eval e AE)) '(4)))
(apply (eval 'twice* GE) '(4))
(apply (list 'compound '(x*) '(plus* x* x*) GE) '(4))
(eval '(plus* x* x*)
  (extend-env-with-new-frame '(x*) '(4) GE))
(eval '(plus* x* x*)
  ((list 'table 'new-frame) (list 'table 'GE)))
> 8
```

其中 apply 中 eval 'twice\* 的过程中，env 使用的是 GE，而不是 AE。本节的 GE 示意图中就可以看出，这里的 GE 是 'twice\* 被 define\* 的环境，而不是 eval 'twice\* 时的环境。这也是我们在环境模型中强调的重要特性。同时，这与非 “strict mode” 下的 Javascript 的闭包 \(closure\) 很相似，但略有区别：

```js
function printWindow () {
    console.log(this.window);
}

printWindow.apply(null, null);
> Window {postMessage: f, blur: f, focus: f, ... }
printWindow.apply({ window: "hello world" }, null)
> "hello world"
```

这里 window 是浏览器中 Javascript Engine 的 GE 中的对象，如果没有给出 this binding，则 this 就是 GE；但如果给出 this binding，则 this 就是给定的 this binding，**它的 lookup 不会被委托 \(delegate\) 到 GE**。环境是否以声明时的外环境为最后委托人是语言的 evaluation model 设计中的取舍点，不同语言会有不同的做法。

### 小结

本节构建了 scheme\* 的解释器，同时也定义 scheme\* language 的语法，它们也是 scheme 解释器和 scheme language 的子集。我们不仅可以用 scheme 来构建 scheme 的解释器，也可以用其它语言来构建 scheme 解释器。有了解释器，我们就可以用它来解决一般问题。

#### 参考

* [Youtube: SICP-2004-Lecture-17](https://www.youtube.com/watch?v=ExeUbrynvNE&index=17&t=0s&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF)
* [MIT6.006-SICP-2005-lecture-notes-17-1](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture19webhan.pdf)
* [MIT6.006-SICP-2005-lecture-notes-17-2](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture19webha2.pdf)





