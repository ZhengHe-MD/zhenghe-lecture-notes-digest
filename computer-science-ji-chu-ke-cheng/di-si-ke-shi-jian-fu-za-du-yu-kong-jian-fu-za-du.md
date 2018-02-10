# 第四课 - 时间复杂度与空间复杂度

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



