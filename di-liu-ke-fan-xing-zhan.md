# 第六课 - 泛型栈

栈是最基本的抽象数据类型之一，这一节探讨了在 C 语言环境下，如何利用无类型指针和内存操作实现一个通用的栈 \(Generic Stack\) — 即可以放进任何类型数据的栈。

### Interfaces

```c
typedef struct {
  void *elems;   	// 指向栈内数据的无类型指针
  int elemSize; 	// 每个数据单元的大小 (字节数)
  int loglength;	// 栈内数据的个数
  int alloclength;	// 为栈分配的动态内存空间能够储存的数据的个数
} stack;

void StackNew(stack *s, int elemSize); 	// 创建栈
void StackDispose(stack *s);			// 回收栈
bool StackEmpty(stack *s);			// 查询栈是否为空
void StackPush(stack *s, void *elemAddr);	// 入栈
void StackPop(stack *s, void *elemAddr);	// 出栈
```

StackNew、StackDispose、StackEmpty、StackPush、StackPop 定义了栈最基本的接口，保证了栈实现的隔离 \(Information Hiding\)

### Implementations

#### StackNew

创建栈时，使用者需要指定栈内数据单元的大小。知道了数据单元的大小后，只需要直接按照相应大小对内存中的字节直接操作即可完成对不同数据类型的支持

```c
void StackNew(stack *s, int elemSize) {
  s->elemSize = elemSize;
  s->loglength = 0;
  s->alloclength = 4;
  s->elems = malloc(4 * elemSize);
}
```

#### StackDispose

StackDispose 只是回收栈中为存储数据动态分配的内存空间，而不是回收栈结构本身。

```c
void StackDispose(stack *s) {
  free(s->elems);
}
```

#### StackEmpty

```c
bool StackEmpty(stack *s) {
  return s->loglength == 0;
}
```

#### StackPush

StackGrow 采用每次倍增栈内数据的存储空间。由于分配存储空间的复杂度为 O\(n\) ，因此这种操作应当尽量避免或者能被均摊 \(amortized\)，而倍增策略在均摊情形下是可以达到 O\(1\) 的，因此这是一种广泛被采用的扩容策略。

StackPush 利用 “指鹿为马” 的黑科技直接将数据源地址中的字节直接拷贝到栈顶。

```c
static void StackGrow(stack *s) {
  s->alloclength *= 2;
  s->elems = realloc(s->elems, s->alloclength * s->elemSize);
}

void StackPush(stack *s, void *elemAddr) {
  if (s->loglength == s->alloclength) {
    StackGrow(s);
  }
  void *destAddr = (char *) s->elems + s->loglength * s->elemSize;
  memcpy(destAddr, elemAddr, s->elemSize);
  s->loglength++;
}
```

#### StackPop

基本可以理解为 StackPush 实现的逆操作。值得一提的是，这里的接口设计要求使用者负责创建和回收存储出栈数据的内存空间，从而减少犯错的几率。

```c
void StackPop(stack *s, void *elemAddr) {
  s->loglength--;
  void *sourceAddr = (char *) s->elems + s->loglength * s->elemSize;
  memcpy(elemAddr, sourceAddr, s->elemSize);
}
```

### 参考

* [Stanford CS107: lecture 6](https://www.youtube.com/watch?v=iyLNYXcEtWE)

* [Github: ZhengHe-MD - lecture codes](https://github.com/ZhengHe-MD/cs107-lecture-codes)



