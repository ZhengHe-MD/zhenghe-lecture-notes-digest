# 第二十三课 - 函数式编程 - Scheme 基础 \(5\)

#### Memory Model by Example

##### 例1：

```scheme
> (cons '(1 2 3) '(4 5 6))
((1 2 3) 4 5 6)
```

内存示意图：

![](/assets/Screen Shot 2018-04-15 at 11.13.33 PM.jpg)

##### 例2：

有两种方式可以输出 **\(\(1 2\) 1 2\)**，但它们的内存模型不一样：

```scheme
> (cons '(1 2) '(1 2))
((1 2) 1 2)
> ((lambda (x) (cons x x)) '(1 2))
((1 2) 1 2)
```

![](/assets/Screen Shot 2018-04-15 at 11.14.05 PM.jpg)

![](/assets/Screen Shot 2018-04-15 at 11.14.27 PM.jpg)

#### variable number of arguments

之前实现过一次 map，接受一个 unary procedure 和一个 list，输出一个 list：

```scheme
> (map car '((1 2) (3 4) (5 6 7)))
(1 3 5)
```

我们称这种 map 为 unary map 实现如下：

```scheme
(define (unary-map fn seq)
  (if (null? seq) ()
      (cons (fn (car seq))
            (unary-map fn
                       (cdr seq)))))
```

但不支持接受一个多元 procedure 和相应多个 lists，输出一个 list：

```scheme
> (map + '(1 2) '(10 20) '(100 400))
(111 422)
> (map * '(1) '(2) '(3) '(4) '(5))
(120)
```

为了实现这种 general-map，我们首先需要能够定义接收任意数量参数的 procedure，即 varargs 语言特性：

```scheme
> (define (bar a b c . d)
    (list a b c d))

> (bar 1 2 3 4 5 6)
(1 2 3 (4 5 6))
```

实现 map

```scheme
(define (map fn first-list . other-lists)
  (if (null? first-list) '()
    (cons (apply fn 
            (cons (car first-list)
                  (unary-map car other-lists)))
          (apply map
            (cons fn
              (cons (cdr first-list)
                    (unary-map cdr other-lists)))))))
```

#### Garbage Collection

当我们在 Scheme 的 REPL 中输入：

```scheme
> (cons "hello" '("world"))
("hello" "world")
```

Scheme 会在内部分配对应的内存，存储 hello 和 world 的指针，以及相应的字符串，在 REPL 打印完毕结果后，这些内存就会被相应地回收。回收的过程大致可以理解成一个递归函数，首先回收 cons 的 car 指针，然后回收 cons 的 cdr 指针，最后回收 construct 本身，其中回收 cons 的 cdr 指针就是回收 cons 的过程，依次递归。

如果使用 define 表达式，这些内存就会被保留：

```scheme
> (define x '(1 2 3))
> (define y (cdr x))
; ...
> (define x '(4 5))
```

当执行完最后一个 define 表达式后，x 指向 '\(4 5\)，y 指向 '\(2 3\)，而为 1 分配的内存没有被引用，到了一定时候，系统的 Garbage Collector 就会启动，回收 1。那么 Garbage Collector 如何判断哪些内存可以回收？

##### reference count \(doesn't work\)

最直观的想法就是为每块内存记录被引用的数量，当引用数量等于 0 时，则可以被回收。但因为 Scheme 支持 set-car!、set-cdr! 这种 mutation 操作，Scheme list 可能出现循环引用，出现一个内存闭环，这个闭环内的数据互相引用，但并没有被实际使用。因此 reference count 的方法无法回收所有不再被使用的内存。

##### mark and sweep

示意图如下所示：

![](/assets/Screen Shot 2018-04-15 at 11.14.54 PM.jpg)

其中 master cons set 存着当前所有被分配的内存，Symbol Set 存着所有环境中的变量。

mark and sweep 分为三个阶段：

1. 把 master cons set 中的所有 cons 标记为可以被回收
2. 遍历 Symbol Set 中的指针，将所有被指到的内存标记为不可被回收
3. 回收所有被标记为可以被回收的内存块 \(sweep\)

#### 参考

* [Stanford-CS107-lecture-23](https://www.youtube.com/watch?v=TJkH1CSHg44&t=0s&list=PL9D558D49CA734A02&index=23)



