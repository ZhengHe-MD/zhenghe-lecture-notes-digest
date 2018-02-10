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



