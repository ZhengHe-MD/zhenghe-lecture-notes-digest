# 第十二课 - 数据修改 （Mutation）

之前的课程中，我们已经讨论数据抽象的几个重要组成部分：构造器 \(constructors\)、选择器 \(selectors\)、操作 \(operations\) 以及契约 \(contract\)。只要按照数据的契约来使用、操作数据，程序就会向着我们预想中的方向运行。为了达到目的，我们需要反复从旧数据中构造新数据。要是我们能够直接修改旧数据，情况又会发生什么样的变化？本节课将引入数据修改 \(Data Mutation\) ，丰富我们的数据抽象思路。

替代模型 \(substitution model\) 假设我们使用函数式编程 \(functional programming\)，没有数据变动和副作用，每段程序都像一个数学函数，无论在什么时候，都准确地将确定的输入映射到确定的输出上。但如果我们引入数据修改，每段程序的输出结果就不仅仅只依赖于输入，还依赖于程序被执行时的环境。举例如下：

```scheme
(define x 10)

(+ x 5)
> 15

(set! x 94)

(+ x 5)
> 99
```

我们将看到

> 引入数据变动使得许多事情变得容易，但会提高犯错的可能性。

### Pair/List Mutation

除了构造器、选择器、操作外，数据抽象可能还包含修改器 \(mutators\)。将数据变动引入到 Scheme 的 pair/list 中，可以得到如下的 pair/list 数据抽象：

```scheme
；constructor
(cons x y)                     ; creates a new pair p

; selectors
(car p)                        ; returns car part of pair
(cdr p)                        ; returns cdr part of pair

; mutators
(set-car! p new-x)             ; changes car pointer in pair
(set-cdr! p new-y)             ; changes cdr pointer in pair
; Pair, anytype -> undef       ; side-effect only
```

其中，set-car! 和 set-cdr! 分别修改 pair 的 car 部分和 cdr 部分指针指向的数据，而且这两种方法只有副作用，没有返回值，因此其返回值是不确定的 \(unspecified\)。

#### 例1：

mutation 给我们编程带来方便的同时引入问题，需要我们格外注意：

```scheme
(define a (list 1 2))
(define b a)

; a ==> (1 2)
; b ==> (1 2)

(set-car! a 10)
; a ==> (10 2)
; b ==> (10 2)
```

由于 a 和 b 指向同一块内存，修改 a 的同时，也会修改 b，如下图所示：

![](/assets/Screen Shot 2018-02-26 at 11.28.23 PM.jpg)

因此我们需要在充分了解 mutation 的基础上加以使用。

#### 例2：

如下图所示修改 x 对应的 list：

![](/assets/Screen Shot 2018-02-26 at 11.28.57 PM.jpg)

可以这样实现：

```scheme
(define x (list 'a 'b))
(set-car! (cdr x) (list 1 2))
```

### Equivalence and Identity

在表示相等关系时，我们常常有这样两个问题：

* a 和 b 是否很像？
* a 和 b 是否是同一个？

如果 a 和 b 很像，那么二者至少表面看起来给人相似的感觉，这就是 equivalence。

如果 a 和 b 是同一个，那么二者实际上指代同一个物体，这就是 identity。

```scheme
; equivalence
; a、b 是否是同一个
(eq? a b)

; identity
; a、b 是否长得像
(equal? a b)
```

### 例1：Stack Data Abstraction

Stack 数据抽象的几个组成部分如下所示：

```scheme
; constructor
(make-stack)         ; returns an empty stack

; selectors
(top stack)          ; returns current top element from a stack

; operations
(insert stack elt)   ; returns a new stack with the element added to the top of the stack
(delete stack)       ; returns a new stack with the top element removed from the stack
(empty-stack? stack) ; returns #t if no elements, #f otherwise

; contract
; if s is a stack, created by (make-stack) and subsequent stack procedures, where i is the
; number of insertions and j is the number of deletions, then
; 1. if j > i :     then it is an error
; 2. if j = i :     then (empty-stack? s) is true, and (top s) and (delete s) are errors
; 3. if j < i :     then (empty-stack? s) is false and (top (delete (insert s val))) = (top s)
; 4. if j <= i:     then (top (insert s val)) = val for any val.
```

#### 实现1：利用 list

```scheme
; constructor
(define (make-stack) nil)

; predicator
(define (empty-stack? stack) (null? stack))

; operations
(define (insert stack elt) (cons elt stack))
(define (delete stack)
    (if (empty-stack? stack)
        (error "stack underflow - delete")
        (cdr stack)))

; selectors
(define (top stack)
    (if (empty-stack? stack)
    (error "stack underflow - top")
    (car stack)))
```

利用 list 直接实现，符合上文中的数据抽象的各个部分。但是我们每次 insert、delete 之后都会生成新的 stack，而并非原来的那个 stack。因此用 eq? 来判断两个 insert 或者 delete 操作前后的 stack 会得到 false。但我们在使用 stack 的过程中，更希望 stack 自始至终是同一个 stack，既符合直觉也能使得操作更加方便。这一切将在引入 mutators 之后得以解决……

#### 实现2：利用 mutators

首先我们需要引入 tag，它有两个好处：

* defensive programming
* 提供 identity --- 试想如果没有 tag，我们将无法做到每次都返回同一个 stack，因为 insert 将改变 list 的 identity，除非我们自己利用 set! 重新将新的 stack 绑定到原来的 name 上。

```scheme
; constructor
(define (make-stack) (cons 'stack nil))

; predicators
(define (stack? stack)
    (and (pair? stack) (eq? 'stack (car stack))))
(define (empty-stack? stack)
    (if (not (stack? stack))
    (error "object not a stack: " stack)
    (null? (cdr stack))))

; mutators
(define (insert! stack elt)
    (cond ((not (stack? stack))
        (error "object not a stack: " stack))
    (else
        (set-cdr! stack (cons elt (cdr stack)))
        stack)))

(define (delete! stack)
    (if (empty-stack? stack)
        (error "stack underflow - delete")
        (set-cdr! stack (cddr stack)))
    stack)

; selector
(define (top stack)
    (if (empty-stack? stack)
        (error "stack underflow - top")
        (cadr stack)))
```

### 例2：Queue Data Abstraction

queue 数据抽象的几个组成部分如下：

```scheme
; constructor
(make-queue)            returns an empty queue

; selectors
(front-queue q)         returns the object at the front of the queue. If queue is empty signals error

; mutators
(insert-queue q elt)    returns a queue with elt at the rear of the queue
(delete-queue q)        returns a queue with the item at the front of the queue removed

; operations/predicators
(empty-queue? q)        tests if the queue is empty
; (queue? q)            tests if the object is a queue

; contracts
; if q is a queue, created by (make-queue) and subsequent queue procedures, where i is the number
; of insertions, j is the number of deletions, and x_i is the i-th item inserted into q, then
; 1. if j > i:          then it is an error
; 2. if j = i:          then (empty-queue? q) is true, and (front-queue q) and (delete-queue q) are errors
; 3. if j < i:          then (front-queue q) = x_(j+1)
```

#### 实现1：没有 mutation

```scheme
; constructor
(define (make-queue) nil)

; predicator
(define (empty-queue? q) (null? q))

; selector
(define (front-queue) q)
    (if (empty-queue? q)
        (error "front of empty queue: " q)
        (car q)))

; mutators
(define (delete-queue q)
    (if (empty-queue? q)
        (error "delete of empty queue: " q)
        (cdr q)))

(define (insert-queue q elt)
    (if (empty-queue? q)
        (cons elt nil)
        (cons (car q) (insert-queue (cdr q) elt))))
```

其中 insert-queue 的时间、空间复杂度皆为 O\(n\)

#### 实现2：mutation

为了减少 insert-queue 的复杂度，我们在引入 tag 的同时，在 queue 中加上队首 \(front \)和队尾 \(rear\) 指针:

![](/assets/Screen Shot 2018-02-26 at 11.31.42 PM.jpg)

```scheme
; helpers, hidden inside abstraction
(define (front-ptr q) (cadr q))
(define (rear-ptr q) (cddr q))
(define (set-front-ptr! q item)
    (set-car! (cdr q) item))
(define (set-rear-ptr! q item)
    (set-cdr! (cdr q) item))


; constructor
(define (make-queue)
    (cons 'queue (cons nil nil)))

; predicator
(define (queue? q)
    (and (pair? q) (eq? 'queue (car q))))
(define (empty-queue? q)
    (if (not (queue? q))
        (error "object not a queue: " q)
        (null? (front-ptr q))))

; selector
(define (front-queue) q)
    (if (empty-queue? q)
        (error "front of empty queue: " q)
        (car (front-ptr q))))

; mutators
(define (delete-queue q)
    (cond ((empty-queue? q)
           (error "delete of empty queue: " q))
          (else
            (set-front-ptr! q
                (cdr (front-ptr q)))
          q)))

(define (insert-queue q elt)
    (let ((new-pair (cons elt nil)))
        (cond ((empty-queue? q)
               (set-front-ptr! q new-pair)
               (set-rear-ptr! q new-pair)
               q)
              (else
               (set-cdr! (rear-ptr q) new-pair)
               (set-rear-ptr! q new-pair)
               q))))
```

这时候，在获得 identity 的同时，insert-queue 的复杂度降到了 O\(1\)

#### 参考

* [Youtube: SICP-2004-Lecture-12](https://www.youtube.com/watch?v=7WlM_bnBEUc)

* [MIT6.006-SICP-2005-lecture-notes-12](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture12webhan.pdf)



