# 第二十课 - 函数式编程 - Scheme 基础 \(2\)

### Scheme 的运行时类型错误检查 \(runtime type checking\)

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

#### Fibonacci

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

#### Flatten

flatten 的功能如下所示：

```scheme
> (flatten '(1 2 3 4)
(1 2 3 4)
> (flatten '(1 (2 3) 4 ((5))))
(1 2 3 4 5)
> (flatten '(1 (2 "3") "4" ((5))))
(1 2 "3" "4" 5)
```

如果用 C 语言来实现这样的函数，我们需要考虑使用链表来作为底层数据结构，来构建算法，最终的代码可能有 50% 的篇幅花在内存管理，50%的篇幅花在构建算法。使用 Scheme 来实现，则可以省去 50% 的内存管理代码。

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



