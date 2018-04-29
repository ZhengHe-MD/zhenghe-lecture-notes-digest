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



