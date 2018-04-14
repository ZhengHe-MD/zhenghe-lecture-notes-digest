# 第二十二课 - 函数式编程 - Scheme 基础 \(4\)

#### generate power set recursively

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

```scheme
(define (ps set)
  (if (null? set) '(())
      (let ((ps-rest (ps (cdr set))))
        (append ps-rest
                (map (lambda (subset)
                  (cons (car set) subset))
                  ps-rest)))))
```

这里的 let-binding 实际上是 lambda 的语法糖：

```scheme
(let ((x _x))
     ((y _y))
  (a x y))
; =>
((lambda (x y)
     (a x y)) _x _y)
```

#### Permutation

```scheme
> (permute '(1 2 3))
((1 2 3) (1 3 2)
 (2 1 3) (2 3 1)
 (3 1 2) (3 2 1))
```

```scheme
(define (permute items)
  (if (null? items) '(())
    (apply append
      (map (lambda (elem)
        (map (lambda (permutation)
               (cons elem permutation))
             (permute (remove items elem)))
        items))))
```

#### Scheme 中原始类型的存储举例

```scheme
> 4
4
> "hello"
hello
> '(1 2 3)
(1 2 3)
> (cons 1 (cons 2 (cons 3 '())))
(1 2 3)
```

4、"hello"、'\(1 2 3\) 在内存中如下图所示：![](/assets/Screen Shot 2018-04-14 at 9.36.29 PM.jpg)而** '\(1 2 3\)** 实际上是** \(cons \(1 \(cons 2 \(cons 3 '\(\)\)\)\) **的语法糖。

#### 参考

* [Stanford-CS107-lecture-22](https://www.youtube.com/watch?v=3LeCydausnk&list=PL9D558D49CA734A02&index=22)



