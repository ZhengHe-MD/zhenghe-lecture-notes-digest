# 第十四课 - 函数的活动记录续、并发

### 函数的活动记录续

#### 例1：上一节课最后一例

```c
int fn() {
    int array[4];
    int i;
    for (i=0; i<=4; i++) {
        array[i] -= 4;
    }
    return 0;
}
```

函数的活动记录如下图所示：

![](/assets/Screen Shot 2018-03-18 at 2.41.14 PM.jpg)

由于 c 语言编译过程没有越界检查，且 array\[3\] + 4 地址处存着执行完 fn 函数之后应该执行的命令地址，即图中虚线箭头指向的地址，但由于 array\[4\] -= 4 的操作使得 saved pc 自减 4，再次指向 fn 函数，因此 fn 将被重复执行，陷入死循环。需要注意的是，本例中谈论的编译过程并不使用于所有情况，不同的编译器会采取不同内存模型，不同的内部参数、返回地址存放方式，本例只是展示可能的一种情况。

#### 例2：

```c
int main() {
    DeclareAndInitArray();
    PrintArray();
}

void DeclareAndInitArray() {
    int array[100];
    int i;
    for (i=0; i<100; i++) {
        array[i] = i;
    }
}

void PrintArray() {
    int array[100];
    for (i=0; i<100; i++) {
        printf("%d\n", array[i]);
    }
}
```

DeclareAndInitArray\(\) 初始化长度为 100 的 array 后，返回 main 函数，但在离开之前它并不会打乱这个 array 里的信息，当 PrintArray 执行的时候，新声明的 array 恰好占据了 DeclareAndInitArray 初始化好的 array 的空间，因此能够将对应的数值打印出来。

#### 例3：printf 如何接受任意数量参数

```c
int printf(const char *control, ...);

printf("%d + %d = %d\n", 4, 4, 8);
```

printf 可以接受任意数量的参数，它的函数原型如第一行代码所示。执行第二行代码时，函数的活动记录如下图所示：

![](/assets/Screen Shot 2018-03-18 at 2.41.43 PM.jpg)

在调用 printf 前，调用者会将参数从右到左压入栈中，printf 开始执行时，它并不知道 saved pc 上面有多少个参数，但它知道它能从第一个 char \* 类型的参数中知道上面还有多少个参数。因此 printf 会根据字符串 control 的内容来按需取用 control 上方的内存信息，这就是接受任意数量参数的函数的工作原理。

调用者将参数从右向左压入函数栈中是一件比较反直觉的做法，如果调用者将参数从左到右压入栈中，函数的活动记录就会如下图所示：

![](/assets/Screen Shot 2018-03-18 at 2.42.04 PM.jpg)

这时候 printf 就无从得知 control 的位置，也就无法从中读出实际参数数量，因此本例也解释了参数入栈顺序的原理。

#### 例4：struct 的多态

```c
struct type {
    int code;
}

struct type-one {
    int code;
    //...
}

struct type-two {
    int code;
    //...
}
```

如果我们的函数需要接受一个结构体，这个结构体可能是两种类似但有所不同的结构体中的一种，比如 ipv4 的 header 与 ipv6 的 header，这时我们就需要在函数中能有某种方式来判断接收到的结构体具体是哪一种。与例3中的原理类似，利用结构体中的第一个成员变量来判断结构体中剩下的信息的种类就可以实现。

### 并发

单核 CPU 能够通过时间片轮转的方式做到多个程序并发运行，从而让计算机的使用者感觉到多个程序正 “同时” 运行。这时候，计算机的内存模型可以概括为下图：

![](/assets/Screen Shot 2018-03-18 at 2.42.26 PM.jpg)

图中有 gcc, make, firefox, lock 四个程序，逻辑上它们分别有自己独立的 stack segment、heap segment 以及 code segment，这些 segments 最终会被内存管理系统映射到内存中的对应地址上，从而实现多个程序在内存中并发运行。

而对于同时运行多个相同的任务的情况，如同时下载两首歌，其下载逻辑，即 code segment 在内存中实际上是共用的，只是 stack segment 和 heap segment 被映射到不同的位置。

#### 参考

* [Stanford CS107: lecture 14](https://www.youtube.com/watch?v=TRfbJIsDBIM&index=14&list=PL08D9FA018A965057&t=225s)



