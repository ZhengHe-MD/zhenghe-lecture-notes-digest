# 第二十一课 - 函数式编程 - Scheme 基础 \(3\)

### map

一种常见的操作就是将一个 sequence 转化成另一个 sequence，其中对 sequence 的每个元素都执行同样的操作，如：

```scheme
> (double-all '(1 2 3 4))
(2 4 6 8)
> (incr-all '(1 2 3 4))
(2 3 4 5)
```

其实这就是我们在其它语言中常见的 map。在 Scheme 中，我们希望这样：

```scheme
> (define (double x) (* x 2))
> (define (incr x) (+ x 1))
; map unary functions
> (map double '(1 2 3 4))
(2 4 6 8)
> (map incr '(1 2 3 4))
(2 3 4 5)
> (map car '((1 2) (4 8 2) (11)))
(1 4 11)
> (map cdr '((1 2) (4 8 2) (11)))
((2) (8 2) ())
; map binary functions
> (map cons '(1 2 8) '((4) () (2 5)))
((1 4) (2) (8 2 5))
> (map + '(1 2) '(3 4) '(6 10))
(10 16)
```

接下来我们尝试实现 map

```scheme
(define (my-unary-map fn seq)
  (if (null? seq) '()
      (cons (fn (car seq)) (my-unary-map fn (cdr seq)))))
```

在进入下一步讨论之前，首先要了解一下 Scheme 中的 apply 和 eval:

eval 接收一个参数，这个参数是程序员输入的原始表达式字符串，之后 eval 会 tokenize、parse 原始表达式字符串，然后再用 evaluator 来处理解析后的 list structure，最终得到输出：

```scheme
> (eval '(+ 1 2 3))
6
```

apply 总是接受两个参数，procedure 的 symbol 和 argument list，apply 会用 cons 把 symbol 和 argument list 合成一个 list 作为输入交给 eval 执行：

```scheme
> (apply + '(1 2 3))
6
> (define (average num-list)
    (/ (apply + num-list)
       (length num-list)))
> (average '(1 2 3 4))
5/2
```

了解了 eval 和 apply 我们可以回顾一下 flatten

```scheme
(define (flatten seq)
  (if (not (list? seq))
    (list seq)
    (apply append
      (map flatten seq))))
```



