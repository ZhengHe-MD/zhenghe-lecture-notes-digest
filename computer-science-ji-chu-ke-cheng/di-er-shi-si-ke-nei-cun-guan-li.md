# 第二十四课 - 内存管理 \(Memory Management\)

### 回顾 register machine

（图1）

在前几课介绍的 register machine 如上图所示，它主要由三个部分组成：

* registers: 用来存储各种 procedures 执行时所需要的参数、环境及输出的结果等数据
* stack: 用来临时存储将被修改的 register 中的数据以便在 subroutine 执行完后恢复环境继续执行
* heap: 用来存储各种各样的具体数据结构的地方。对于 scheme 来说就是存储所有 list 的地方，无论是表达式还是数据都是 list。

本节介绍的就是 heap 及其所依赖的技术。

### Abstraction

#### Vector

内存支持数据的随机存取，即可以在常数级别的时间复杂度内存取内存中任意位置上的数据，这可以被抽象为 Vector 模型，如下图所示：

（图2）

其中在 Scheme、Register Machine 中相应的操作也如上图所示。

#### Stack

我们也很容易使用 Vector 来实现 Stack：用一个 stack pointer \(SP\) 指向当前 stack 的顶部，每当 save 的时候就往 vector 增大的方向移动 SP；每当 restore 的时候就往 vector 减小的方向移动 SP：

```scheme
(save <reg>)
; becomes
(assign sp (op +) (reg sp) (cons 1))
(perform (op vector-set!) (reg stack)
          (reg sp) (reg <reg>))


(restore <reg>)
; becomes
(assign <reg> (op vector-ref) (reg stack) (reg sp))
(assign sp (op - ) (reg sp) (cons 1))
```

#### Heap

在 register machine 中，heap 就是一堆 cons，它们分别由 car 和 cdr 构成。为了支持 cons，我们可以使用两个平行等长的 vector 来模拟，第一个 vector 叫作 the-cars，第二个 vector 叫作 the-cdrs，它们共用 base index，如下图所示：

（图3）

##### car & cdr

cons 的 selectors 操作，即 car 和 cdr 如下文所示：

```scheme
; CAR
(assign <reg> (op car) <pair>)
; becomes
(assign <reg> (op vector-ref)
              (reg the-cars) <pair>))

; CDR
(assign <reg> (op cdr) <pair>)
; becomes
(assign <reg> (op vector-ref)
              (reg the-cdrs) <pair>)
; where <pair> is a cell offset into the-cars & the-cdrs vectors
```

##### cons

heap 不仅要支持 cons 数据的获取，还需要支持 cons 的分配与回收，即 heap 的内存管理。首先我们需要新增一个 register --- free，它指向当前尚未分配的最近一个 cell，当分配操作被执行时，free 指针指向的 cell 被分配出去，free 指针则指向下一个尚未分配的 cell，整个过程示意图如下所示：

（图4）

cons 的执行过程如下所示：

```scheme
; CONS
(assign <rename> (op cons) <val-1> <val-2>)
; becomes
(perform (op vector-set!)
         (reg the-cars) (reg free) <val-1>)
(perform (op vector-set!)
         (reg the-cdrs) (reg free) <val-2>)
(assign <regname> (reg free))
(assign free (op +) (reg free) (const 1))
```

那么我们如何区别 cons cell 中储存的不同类型的数据呢？比如 null、integer、boolean、指向其它 cell 的指针等等。我们可以拿出 \(car cell\)、\(cdr cell\) 中数据的一部分 bits 来存储数据的类型，剩下的 bits 用于存取具体的数据内容，如下图所示：

（图5）

###### 举例：

```scheme
(define a (list 4 7 6))
(cons a a)
```

利用刚才的 vector 抽象，我们足以支持以上语句中的 list 数据创建，如下图所示：

（图6）

##### cell 的回收：Garbage Collection

如果内存足够大，我们可以不断地分配 cells，但我们知道内存空间无论再怎么大也是有限度的，当空间不足时，我们就需要把不用的 heap 空间回收，即所谓垃圾回收 \(Garbage Collection\)。

##### 垃圾生成 \(Creation of Garbage\)

垃圾就是在当下之前分配的、直到程序运行结束时都不可能再被使用到的内存空间。一个简单的垃圾生成的例子如下图所示：

（图7）

##### 垃圾检测 \(Detection of Garbage\)

与程序之后运行有关的数据都存储在 registers、stack 以及 global environment 中，因此只要把它们中指向的 heap cons cells，以及这些 cells 指向的其它 cells 保留，剩下的没有被引用到的 cells 就是垃圾。

##### 垃圾回收法1 --- Mark/Sweep Garbage Collection

Mark & Sweep 顾名思义，分为 Mark 和 Sweep 两个阶段。Mark 阶段 Garbage Collector 会顺着 registers、stack 和 global environment 出发，把所有被直接或间接引用到的 cells 都 mark 成 1；Sweep 阶段在 Mark 阶段完成后，遍历一遍被分配的 cells，把所有没有被 mark 成 1 的 cells 全部回收。接着刚才的 the-cars 和 the-cdrs 模型，我们新增一个 the-marks vector，就可以分别完成 mark 和 sweep，举例如下图所示：

（图8）

Mark 阶段：从 registers、stack 和 global environment 中的 c 出发，找到直接引用 4 以及间接引用 0、1，于是它们被 mark 成 1。

Sweep 阶段：将没有被 mark 成 1 的 2、3 回收。

Mark/Sweep 的示例代码如下：

```scheme
; Mark Phase
(define (mark object)
  (cond ((not (pair? object)) #f)
        ((not (= 1 (vector-ref the-marks object)))
         (vector-set! the-marks object 1)
         (mark (car object))
         (mark (cdr object)))))
; For a pair, "object" is an integer offset denoting the cell location

; Sweep Phase
(define (sweep i)
  (cond ((not (= i size))
         (cond ((= 1 (vector-ref the-marks i))
                (vector-set! the-marks i 0))
               (else (set-cdr! (int-to-pointer i) free)
                     (set! free (int-to-pointer i))))
         (sweep (+ i 1)))))

(define (gc)
  (mark root)
  (sweep 0))
```

##### 垃圾回收法2 --- Stop & Copy Garbage Collection

使用 Stop & Copy 策略时，Garbage Collector 将可支配内存空间分为两半，只将一半用于分配。当用于分配的一半分配完后，同样从 registers、stack 和 global environment 出发遍历所有直接引用和间接引用的空间，将其中的信息复制到另一半未分配的空间中去，并保持它们之间的引用关系。相较于 Mark & Sweep 策略，Stop & Copy 需要两倍内存空间，但后者可以利用复制的过程来做碎片整理。

#### 小结

垃圾回收最早需要程序员手动完成，但由于这个过程即使是素质过硬的程序员也在所难免会出现错误，这种错误难以检查、发现，将它交给程序管理，确实是一种进步。但作为站在巨人肩上的程序员，了解一下你为什么能站在巨人的肩膀上，为什么能写着在天上飞的语言还是很有必要的。

#### 参考

* [Youtube: SICP-2004-Lecture-24](https://www.youtube.com/watch?v=YA1lB8hFYUI&index=24&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&t=0s)



