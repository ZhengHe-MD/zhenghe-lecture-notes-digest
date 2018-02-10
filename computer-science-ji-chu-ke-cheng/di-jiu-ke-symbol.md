# 第九课 - Symbol

### Symbol

* Say your favorite color — value associated with a name \(symbol\)

* Say "your favorite color" — symbol

```scheme
; define a symbol
(define alpha 27)
(quote alpha)
'alpha
;; ' is a shorthand for quote

; retrieve the value associated with symbol
alpha

; reference the symbol
(quote alpha)
'alpha

; operations
(symbol? (quote alpha)) ; test whether an object is a symbol
(eq? 'alpha 'alpha)     ; test equality of two symbols
```

由前面的章节，我们知道 Scheme 在对表达式解析时，会先判断表达式类型。比如 lambda 表达式，那么 Scheme 的 evaluator 会认为这是一种特殊表达式，它的评价方式是创建一段对应的程序，然后返回指向该程序的指针，这时候我们就会看到 \#\[compound-…\] 这样的打印输出结果。

![](/assets/evaluate-lambda.jpg)

同样的道理，Scheme 如果遇到 quote 表达式，它也会认为这是一种特殊表达式，它的评价方式就是为表达式中的第二个子表达式创建内部表示 \(internal representation\) 并返回，这时候 Scheme 会把对应的内部表示打印出来 — 即 beta。

![](/assets/evaluate-quote.jpg)

我们可以像使用其他原始类型数据一样使用 Symbol

```scheme
(list (quote delta) (quote delta))
```

但 Scheme 的解释器内部实际上会记住过往的 symbols，因此解释器内部不会存在两个一模一样的 symbol，即这个 symbol 是全局唯一的。因此上文中的表达式实际上在解释器内部表示如下图所示：

![](/assets/scheme-expression-to-list.jpg)然后再对这些列表进行评价。而评价 quote 这种特殊表达式时，就将 quote 后面的整个列表返回而不作任何额外评价，因此最后打印出来的返回信息就是该表达式本身。更多例子举例如下：

```scheme
(define x 20)

(+ x 3) 				; => 23
'(+ x 3)				; => (+ x 3)
(list (quote +) x '3)	; => (+ 20 3)
(list '+ x 3) 			; => (+ 20 3)
(list + x 3)			; => ([procedure #...] 20 3)
```

### Symbol 提高 Scheme 语言表达力

#### 例：Symbolic Derivatives

```scheme
; (deriv <expr> <with-respect-to-var>) ==> <new-expr>
;
; syntax
; Expr = SimpleExpr | CompoundExpr
; SimpleExpr = number | symbol
; CompoundExpr = pair< (+|*), pair<Expr, pair<Expr, null> >>

; usage
; (deriv '(+ x 3) 'x) 			; => 1
; (deriv '(+ (* x y) 4) 'x)		; => y
; (deriv '(* x x) 'x) 			; => (+ x x)

; implementation
(define simple-expr? (lambda (e)
  (not (pair? e))))

(define deriv (lambda (expr var)
  (if (simple-expr? expr)
      (if (number? expr) 0
          (if (eq? expr var) 1 0))
      (if (eq? (car expr) '+)
          (list '+
                (deriv (cadr expr) var)
                (deriv (caddr expr) var))
          <handle product expression>
      )
  ))
)

; 缺点
; 1. 可读性差 			 <- 没有明确的函数名告诉读者每段程序在做什么
; 2. 对新操作的扩展性差 		 <- 因为目前的代码假设只有两种操作，利用嵌套 if 语句来完成
; 3. 对表达式的表示形式不可变       <- 如果我们要改变表达式的表示形式，比如 '(x + 3)，代码将发生巨大改变，因为我们依赖了 list 结构，以及它的选择器, car, cdr, cadr ... 而没有抽象出 Expr 这种抽象数据类型。

; 改进
; 1. 使用 cond 而不是嵌套if
(define sum-expr? (lambda (e)
  (and (pair? e) (eq? (car e) '+))))
(define variable? (lambda (e)
  (and (not (pair? e)) (symbol? e))))
; 2. 使用数据抽象
(define make-sum (lambda (e1 e2)
  (list '+ e1 e2)))
(define addend (lambda (sum) (cadr sum)))
(define augend (lambda (sum) (caddr sum)))

; deriv
(define deriv (lambda (expr var)
  (cond
    ((number? expr) 0)
    ((variable? expr) (if (eq? expr var) 1 0))
    ((sum-expr? expr)
      (make-sum (deriv (addend expr) var)
                (deriv (augend expr) var)))
    ((product-expr? expr)
      <handle product expression>)
    (else
       (error "unknown expression type" expr))
   )
))

; 此时 (deriv (make-sum 'x 'y) 'x) => '(+ 1 0)，但我们希望得到的是约减过的表达式 '1，为了得到后者
(define make-sum
  (lambda (e1 e2)
    (cond ((number? e1)
           (if (number? e2)
               (+ e1 e2)
               (list '+ e1 e2)))
          ((number? e2)
           (list '+ e2 e1))
          (else (list '+ e1 e2)))))
```

### 参考

* [Youtube: SICP-2004-Lecture-9](https://www.youtube.com/watch?v=1SwPKtAIEwA&index=9&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF)

* [MIT6.006-SICP-2005-lecture-notes-9](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture9webhan.pdf)



