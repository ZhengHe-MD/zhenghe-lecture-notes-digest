# 第十三课 - 编译的链接过程



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



