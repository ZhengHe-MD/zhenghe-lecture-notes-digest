# 第二十课 - 函数式编程 - Scheme 基础 \(2\)

### 运行时类型错误检查 \(runtime type checking\)

上节末尾引入了 sum-list procedure:

```scheme
(define (sum-list num-list)
  (if (null? num-list) 0
      (+ (car num-list)
         (sum-list (cdr num-list)))))

> (sum-list '(1 2 3 4 5))
15
> (sum-list '())
0
```

表达式存在类型错误时，如：

```scheme
> (sum-list '("hello" 1 2 3 4 5))
string cannot be + with number
```

Scheme 会正常执行这个语句，直到最后执行 \(+ "hello" 15\) 时发现 "hello" 与 15 并不能直接相加，进而抛出类型不匹配的错误。与 C 等静态类型语言不同，Scheme 在编译的过程中主要做语法解析而没有做类型检查，只有到运行时环境下具体执行类型不匹配的操作时才抛出错误。

```scheme
> (if (zero? 0) 4
      (+ "hello" 4.5 '(8 2)))
4
```

如上表达式，if 语句的 alternative 表达式可以顺利通过语法解析，但在运行时由于在逻辑上它不会被执行，因此整条语句在解释器 evaluate 的时候不会抛错。

### Recursion in Scheme

用 recursion 可以以逻辑十分清晰的方式构建程序

#### fibonacci

```scheme
(define (fib n)
  (if (zero? n) 0
      (if (= n 1) 1
          (+ (fib (- n 1))
             (fib (- n 2))))))
; 或者
(define (fib n)
  (if (or (= n 0)
          (= n 1)
      n
      (+ (fib (- n 1))
         (fib (- n 2)))))
> (fib 0)
0
> (fib 1)
1
> (fib 2)
1
> (fib 3)
2
```

#### flatten

flatten 的功能如下所示：

```scheme
> (flatten '(1 2 3 4)
(1 2 3 4)
> (flatten '(1 (2 3) 4 ((5))))
(1 2 3 4 5)
> (flatten '(1 (2 "3") "4" ((5))))
(1 2 "3" "4" 5)
```

如果用 C 语言来实现这样的函数，我们首先需要考虑使用链表来作为底层数据结构，并且链表中的元素需要兼容不同的数据类型。有了链表后再以此为基础来构建算法，最终的代码可能有 50% 的篇幅花在内存管理，50%的篇幅花在构建算法。使用 Scheme 来实现，则可以省去内存管理代码，直接把时间花在算法的构建上。

```scheme
; '()
; '(1 ...)
; '((1 2) ...)

(define (flatten sequence)
  (cond
    ((null? sequence) '())
    ((list? (car sequence))
      (append (flatten (car sequence))
              (flatten (cdr sequence))))
    (else (cons (car sequence)
                (flatten (cdr sequence))))))
```

#### sorted?

检查 num-list 是否按升序排列

```scheme
> (sorted? '(1 2 2 4 7))
#t
> (sorted? '(1 0 4 7 10))
#f
```

当 num-list 的元素少于两个时，这个 num-list 已经按升序排列完成；当元素大于或等于两个时，这个 num-list 需要同时满足：

* 第一个元素小于或等于第二个元素
* num-list 去掉第一个元素剩下的元素按升序排列好

```scheme
; '()
; '(1)
; '(x y ...)
(define (sorted? num-list)
  (or (< (length num-list) 2)
      (and (<= (car num-list)
               (cadr num-list))
           (sorted? (cdr num-list)))))
```

#### general sorted?

与前几课介绍的 C 语言的 general sort 函数相似，我们也希望 sorted? procedure 可以接受类似函数指针一样的东西来指导排序的过程。

```scheme
> (sorted? '(1 2 3 4) <=)
#t
> (sorted? '("a" "b" "d" "c") string<?)
#f
```

在 Scheme 中不需要 procedure 指针，procedure 本身可以被作为参数传入别的 procedure 中：

```scheme
(define (sorted? seq comp)
  (or (< (length seq) 2)
      (and (comp (car seq)
                 (cadr seq))
           (sorted? (cdr seq) comp))))
```

#### 参考

* [Stanford-CS107-lecture-20](https://www.youtube.com/watch?v=onKR7ICXacQ&index=20&t=2s&list=PL9D558D49CA734A02)



