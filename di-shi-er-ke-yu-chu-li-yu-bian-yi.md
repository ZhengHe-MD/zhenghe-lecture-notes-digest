# 第十二课 - 编译预处理

一段 c 代码编译的过程按顺序实际上分为三个步骤：预处理 \(preprocess\)、编译 \(compile\) 以及链接 \(link\)，如下图所示：

![](/assets/Screen Shot 2018-03-04 at 11.23.16 PM.jpg)

预处理可以理解成一个文本替换过程，它可以帮你将 \#define 的内容进行文本替换，预处理完的文本将传递给编译器；编译器将 c 代码编译成汇编语言，接着将汇编代码传递给链接器 \(linker\)；链接器将引用到的其它代码和编译好的代码合并成可执行程序，即编译过程的最终结果。

### 预处理阶段

##### 例1：\#define

```c
// preprocessor_test_1.c
#define kwidth 40
#define kheight 80
#define kPerimeter 2*(kwidth + kheight)

int main() {
    int area = kwidth * kheight;
    return kPerimeter;
}
```

输入以下命令执行预处理过程：

```bash
$ clang -E preprocessor_test_1.c
```

得到的预处理结果如下：

```c
int main() {
    int area = 40 * 80;
    return 2*(40 + 80);
}
```

预处理器会从上到下扫描输入文件，读到 **\#define **时，记录下 token 及其对应的文本内容，当它在代码中遇到相应的 token，如 **kwidth**、**kheight** 后，就会直接将对应的文本替换到相应位置，同时把 **\#define** 语句去除。

##### 例2：宏 \(macro\)

```c
// preprocessor_test_2.c
#define MAX(a,b) (((a) > (b)) ? (a) : (b))

int main() {
    return MAX(3, 4);
}
```

输入同样的命令执行预处理过程，得到的结果如下：

```c
int main() {
    return (((3) > (4)) ? (3) : (4));
}
```

预处理器也支持宏 \(macro\) 定义，它类似函数但没有类型定义，这是带有参数的文本替换。

##### 例3：宏 - 文本替代详解

```c
// preprocessor_test_3.c
#define MAX(a,b) (((a) > (b)) ? (a) : (b))

int main() {
    return MAX(3, "hello world");
}
```

显然，这段代码不是合法的 c 代码，int 无法与 char\* 比较。虽然编译无法通过，但预处理过程依然可以执行，得到的结果如下：

```c
int main() {
    return (((3) > ("hello world")) ? (3) : ("hello world"));
}
```

也印证了预编译过程仅仅做了文本替代，并没有涉及其它部分的功能。

##### 例4：

```c
#define NthElementAddr(base, elemSize, index) \
    ((char *)base + index * elemSize)

void *VectorNth(vector *v, int position) {
    assert(position >= 0);
    assert(position < v->loglength);
    return NthElemAddr(v->elems, v->elemSize, position);
}
```

在实现抽象数据类型 Vector 时，我们需要多次使用指针算术来计算一段内存的第 n 个元素地址，这种常用的计算模式就可以抽象成宏或者利用 helper function 来完成，本例就是利用宏来实现抽象。预处理的结果不再展开。

##### 例5：

实际上，我们在防御性编程中，经常使用到的 assert，也是定义在 assert.h 中的一个宏，它的完整实现可以在类似 **/usr/include/assert.h** 路径中查到，这里简单介绍一下它的简单实现：

```c
#ifdef NDEBUG
    #define assert(cond) (void)0
#else
    #define assert(cond) \
        (cond) ? ((void) 0) :
            fprintf(stderr, "..........."), exit(0)
#endif
```

当 NDEBUG 为 true 时，assert 被替换为 \(\(void\)0\)，即 no-op，编译时会在优化的过程中去除。



**需要注意的是：在使用宏时，要注意它只做文本替换，程序员需要警惕替换的结果可能造成的问题。**

##### 例6：使用宏可能出现效率问题

```c
#define MAX(a,b) (((a) > (b)) ? (a) : (b))

int main() {
    return MAX(fib(100), fact(4000));
}
```

会被预处理成

```c
int main() {
    return (fib(100) > fact(4000)) ? fib(100) : fact(4000);
}
```

其中 fib\(100\) 与 fact\(4000\) 中的较大一方将被执行两次。

##### 例7： 使用宏可能出现逻辑问题

```c
#define MAX(a,b) (((a) > (b)) ? (a) : (b))

int main() {
    return MAX(m++, n++);
}
```

会被预处理成

```c
int main() {
    return (((m++) > (n++)) ? (m++) : (n++));
}
```

显然，m 和 n 有一个会自增两次。

##### 例8：\#include

```c
#include <stdio.h>
#include <assert.h>
#include "vector.h"
```

当预处理器遇到 **\#include **的时候，它会去寻找对应的文件，用该文件的所有内容代替 **\#include **语句。当文件名用尖括号包裹的时候，预处理器会从系统路径中寻找，如 /usr/bin/include 或者 /usr/include 等等；当文件名用双引号包裹的时候，预处理器会从当前工作目录下寻找。同时，**\#include **语句替换是递归的，如果在读取的文件中读到 **\#include **，预处理器将递归地去读取对应的文件。

> 一般情况下，我们只在头文件 \(\*.h\) 中声明函数原型、结构体、全局变量、宏等信息，而没有相关实现。原因在于编译后的汇编代码实际上没有声明，只有实现，因此这些声明会在编译后消失。

#### 参考





