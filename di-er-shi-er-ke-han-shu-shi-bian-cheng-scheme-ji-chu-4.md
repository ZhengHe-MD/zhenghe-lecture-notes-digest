# 第二十二课 - 函数式编程 - Scheme 基础 \(4\)

### generate power set recursively

```scheme
; ps => power-set
> (ps '(1 2 3))
(() (2) (3) (2 3)          ; 不含 1 的集合
  (1) (1 2) (1 3) (1 2 3))  ; 含 1 的集合

> (ps '())
(())


(define (ps set)
  (if (null? set) '(())
      (append (ps (cdr set))
              (map (lambda (subset)
                (cons (car set) subset))
                (ps (cdr set))))))
```

上面的实现非常的精简，就是把一个 set 的 power-set 看作是两部分的集合：

* 不含 set 第一个元素的 power-set \(记为 ps-rest\)
* 包含 set 第一个元素的 power-set \(记为 ps-all\)

其中 ps-all 可以看作是第一个元素与 ps-rest 之间的每个元素的分别取合的集合。但以上实现有一个缺点，就是 **\(ps \(cdr set\)\)** 被执行了两次:  




