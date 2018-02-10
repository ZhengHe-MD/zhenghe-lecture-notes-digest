# 第四课 - 时间复杂度与空间复杂度

### 例1：Factorial

#### Recursive Version

```scheme
(define fact (lambda (n)
  (if (n = 1) 
     1
     (* n (fact (- n 1))))))
; 递归版本
; (fact 4)
; (if (= 4 1) 1 (* 4 (fact (- 4 1))))
; (* 4 (fact 3))
; (* 4 (if = 3 1) 1 (* 3 (fact (- 3 1))))
; (* 4 (* 3 (fact 2)))
; (* 4 (* 3 (if (= 2 1) 1 (* 2 (fact (- 2 1))))))
; (* 4 (* 3 (* 2 (fact 1))))
; (* 4 (* 3 (* 2 (if (= 1 1) 1 (* 1 fact (-1 1))))))
; (* 4 (* 3 (* 2 1)))
; (* 4 (* 3 2))
; (* 4 6)
; 24

; 空间复杂度: θ(n)
; 时间复杂度: θ(n)
```

#### Iterative Version

```scheme
; 迭代版本
(define ifact-helper (lambda (product count n)
    (if (> count n)
        product
        (ifact-helper (* product count) (+ count 1) n))))
; (ifact 4)
; (ifact-helper 1 1 4)
; (if (> 1 4) 1 (ifact-helper (* 1 1) (+ 1 1) 4))
; (ifact-helper 1 2 4)
; (if (> 2 4) 1 (ifact-helper (* 1 2) (+ 2 1) 4))
; (ifact-helper 2 3 4)
; (if (> 3 4) 2 (ifact-helper (* 2 3) (+ 3 1) 4))
; (ifact-helper 6 4 4)
; (if (> 4 4) 6 (ifact-helper (* 6 4) (+ 4 1) 4))
; (ifact-helper 24 5 4)
; 24

; 空间复杂度: θ(1)
; 时间复杂度: θ(n)
```

### 例2：Fibonacci

```scheme
; Fibonacci
(define fib
  (lambda (n)
    (cond ((= n 0) 0)
          ((= n 1) 1)
          (else (+ (fib (- n 1))
                   (fib (- n 2)))))))
; (fib 4)
; (+ (fib 3) (fib 2))
; (+ (+ (fib 2) (fib 1)) (+ (fib 1) (fib 0)))
; (+ (+ (+ (fib 1) (fib 0) 1) (+ 1 0)))
; (+ (+ (+ 1 0) 1) 1)
; (+ (+ 1 1) 1)
; (+ 2 1)
; 3

; 空间复杂度: θ(n)，原因在于计算的过程是深度优先遍历递归树
; 时间复杂度: O(2^n),
```

### 例3：pow\(a, b\)

#### Recursive Version

```scheme
(define my-expt
  (lambda (a b)
    (if (= b 0)
        1
        (* a (my-expt a (- b 1))))))
        
; 空间复杂度: θ(n)
; 时间复杂度: θ(n)
```

#### Iterative Version

```scheme
(define exp-i (lambda (a b) (exp-i-help 1 b a)))

(define exp-i-help
  (lambda (prod count a)
    (if (= count 0)
        prod
        (exp-i-help (* prod a) (- count 1) a))))
; 空间复杂度: θ(1)
; 时间复杂度: θ(n)
```

#### Faster Recursive Version

```scheme
(define fast-exp lambda (a b)
  (cond (= b 1) a)
  		((even? b) (fast-exp (* a a) (/ b 2)))
  		(else (* a (fast-exp a (- b 1)))))
; 空间复杂度: θ(log(n))
; 时间复杂度: θ(log(n))
```

### 例4：杨辉三角 （计算 n 行 j 列数值）

```scheme
;           1
;         1   1
;       1   2   1
;     1   3   3   1
;   1   4   6   4   1
; 1   5  10   10  5   1

; 传统方法计算

(define pascal
  (lambda (j n)
    (cond ((= j 0) 1)
          ((= j n) 1)
          (else (+ (pascal (- j 1) (- n 1))
                   (pascal j (- n 1))))))))
; 时间复杂度: 指数增长
; 空间复杂度: θ(n)

; 使用组合公式计算
(define pascal
  (lambda (j n)
    (/ (fact n)
       (* (fact (- n j)) (fact j)))))
; 时间复杂度: θ(n)
; 空间复杂度: θ(n)

(define pascal
  (lambda (j n)
    (/ (ifact n)
       (* (ifact (- n j)) (ifact j)))))
; 时间复杂度: θ(n)
; 空间复杂度: θ(1)

; 将分子分母公共部分约分，直接计算
(define pascal
  (lambda (j n)
    (/ (help n 1 (+ n (- j) 1))
       (help j 1 1))))
(define help
  (lambda (k prod end)
    (if (= k end)
        (* k prod)
        (help (- k 1) (* prod k) end))))
; 时间复杂度: θ(n)
; 空间复杂度: θ(1)
```

### 小结

本课目的在于利用替代模型展开不同程序的计算过程，从而直接体会不同程序的设计方式对复杂度 \(order of growth\) 的影响，有助于在写代码过程中下意识书写更高效的代码 \(我：在不失可读性的情况下\)。

### 参考

[MIT6.006-SICP-2005-lecture-notes-4](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture4webhand.pdf)



