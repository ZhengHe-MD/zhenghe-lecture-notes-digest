# 第十三课 - 编译的链接过程

本节用多个例子来介绍链接过程：

### 例1

```c
// mem.c
#include <stdlib.h>    // (1)
#include <stdio.h>     // (2)
#include <assert.h>    // (3)

int main() {
    void *mem = malloc(400);
    assert(mem != NULL);
    printf("Yay! \n");
    free(mem);
    return 0;
}
```

以上 mem.c 文件经过预处理、编译阶段后，会生成 mem.o 文件，这时候所有用户自定的常数、宏被预处理器替换，用户编写的代码被编译成机器码，但还缺少标准库的实现，即 stdio.h, stdlib.h 中定义的函数实现。这时候就需要链接过程来完成标准库 .o 文件与 mem.o 文件的打包工作，如下图所示：

（图1）

执行以下命令即可编译 mem.c

```bash
$ gcc mem.c
```

接下来我们通过分别注释掉 mem.c 前三个 \#include 语句来窥探编译的链接过程。

##### 注释 \#include &lt;stdio.h&gt;

我们发现 mem.c 编译成功，原因在于 gcc 会自动猜测 printf 的参数类型和返回值类型。如上面的代码中，main 函数调用 printf 并传入参数 "Yay! \n"，因此 gcc 猜测 printf 的参数为 char\* 类型，返回值为 int 型 （默认），而经过链接阶段，printf 函数有了相应的实现代码，并且类型符合，因此整个编译过程没有出现问题。

值得一提的是，头文件 \*.h 对于编译器来说仅仅提供验证函数、结构体等个体在使用过程中的合法性，与最终的实现毫无关系，最终的实现取决于链接过程中打包进来的具体实现，而每个编译器的工具箱中都有标准库实现，因此上述编译过程能够顺利完成。

##### 注释 \#include &lt;stdlib.h&gt;

与上一例同理，mem.c 可以编译成功。不同的是，malloc 的函数原型返回值是指针，而这里在编译的过程中默认返回值是整数，这时候同样会出现警告 \(由于原课程是在 08 年发布的，因此 gcc 的输出可能不一样，我们这里以理解编译的链接过程为目的\)，但依然可以编译通过，因为在链接过程中，打包进来的标准库函数 malloc, free 的实现与代码中的使用方式是相符的，它们与编译器的猜测不符，但最终代码仍然可以编译成功。

##### 注释 \#include &lt;assert.h&gt;

在上一课，我们提到 assert 并不是函数，而是宏，它在预处理阶段将被对应的文本替换，如果我们将 assert.h 注释，assert 不会被替换成对应的宏；编译过程虽然会提出警告，但仍然可以进入下一步；最终的链接过程，连接器找不到 assert 对应的实现，因此抛错拒绝编译通过。

##### 函数原型声明

实际上，函数原型 \(prototype\) 是函数的调用者 \(caller\) 与执行者 \(callee\) 之间的协议，函数的调用者在正式调用函数前，会把参数放在函数栈相应的位置；函数的执行者会默认函数的调用者已经把规定好的参数放在相应的位置来执行函数体。

### 例2

```c
// int strlen(char *s, int len);

int main() {
    int num = 65;
    int length = strlen((char *) &num, num);
    printf("length = %d\n", length);
    return 0;
}
```

编译过程中，当编译器遇到 strlen 时，自动猜测 strlen 的类型为 int strlen\(char \*, int\)，只会发出警告，编译可以顺利通过 \(注：在新的 clang 和 gcc 上，编译会抛错，但如果 uncomment 第一行的 strlen 原型声明，则编译通过\)。编译器中的标准库实现 \(.o 文件\) ，因为本身已经是汇编语言，操作对象都是无类型的内存块，因此在这个阶段它们无法判断调用者传进来的参数是否符合要求，只能完全信任它。这时候，函数栈的部分示意图如下所示：

（图1）

从图中可以看出，虽然调用者放了两个参数在内存中，但是函数执行者 strlen 实际只用一个 char\* 参数，因此它的实际使用范围在虚线一下。在 little-endian 机器中，代码执行后会打印 1；在 big-endian 机器中，代码执行后会打印 0。

### 例3

```c
// memcmp 实际的函数原型
// int memcmp(void *v1, void *v2, int size);

int memcmp(void *v1);

int main() {
    int n = 17;
    int m = memcmp(&n);
}
```

代码中的 memcmp 原型声明虽然和实现不一致，但这不影响编译的顺利进行，但运行时，memcmp 的 caller --- main 函数只为 memcmp 准备了一个参数，也就是 &n，而 memcmp 本身的实现却需要三个参数，因此它会认为 saved pc 往上 12 字节都是自己的参数，这时候运行就可能出现严重的问题。具体的函数栈如下图所示：

（图3）

### 例4

进入例4之前，先了解两种代码崩溃时抛出的错误

#### Segmentation Fault

运行时，在内存中有 4 个 segments， data segment、stack segment、heap segment and code segment。这些 segments 在内存中都被分配到某一段内存地址范围，当你的代码尝试 dereference 一个不属于这 4 个 segments 的地址时，就会出现 segmentation fault。比如 \*\(NULL\)。

#### Bus Error

运行时，为了简单硬件会将 short、int 等数值类型永远放在指定地址 \(2 或 4 的倍数\)，一旦它发现代码中存在从非指定地址去读取这些数值类型，就会抛出 Bus Error。

接下来看例子

```c
int main() {
    int i;
    int array[4];
    for (i=0; i<=4; i++) {
        array[i] = 0;
    }
    return 0;
}
```

可以看出代码中的 for 循环多了一次，而这多了的一次可以造成程序陷入死循环 \(在 clang-8000.0.42.1 上没有重现 \)，具体内存模型如下图所示：

（图4）

另外两个例子与此类似，这里只给出相关代码段

```c
// 可能造成死循环也可能不，看系统的 endian
int main() {
    int i;
    short array[4];
    for (i=0; i<=4; i++) {
        array[i] = 0;
    }
    return 0;
}
```

```c
// 造成死循环，修改了 saved pc
int main() {
    int array[4];
    int i;
    for (i=0; i<=4; i++) {
        array[i] -= 4;
    }
    return 0;
}
```



