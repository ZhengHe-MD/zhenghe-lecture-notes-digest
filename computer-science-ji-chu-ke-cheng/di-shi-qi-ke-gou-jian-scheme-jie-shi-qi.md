# 第十七课 - 构建 Scheme 解释器

我们用编程语言写的每一句表达式，都有它的含义，将表达式推导转化成它的实际含义的过程，被称为 evaluation。而实现这个推导转化过程的程序，就是解释器（interpreter）。

### 解释器的基本构造

解释器通常由 Lexical Analyzer、Parser、Evaluator & Environment、Printer 四个部分组成，它们的关系如下图所示：

\(图1\)

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

（图2）

eval 过程通过判断表达式的第一个 token 的类型，来决定要对其进行什么样的操作 --- 如果是 number，就直接返回对应数值，如果是 plus\*，则将表达式交给 eval-sum 去递归推导转化成最终结果。

上文代码利用数据驱动及防御式的编程方式组织 eval 过程，同时利用递归的方式将复杂表达式一层一层剥开，直到最简单的表达式，然后再将结果一层一层组合，最终推导得到计算表达式的结果。这就是一个最简单的 evaluator。

#### names

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

在引入

