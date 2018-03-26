# 第十六课 - 信号量与多线程

### Reader and Writer

Writer 将往 buffer 写入 40 个字节，Reader 将读取 Writer 写入 buffer 的 40 字节。

```c
char buffer[8];

int main() {
    InitThreadPackage(false);
    ThreadNew("Writer", Writer, 0);
    ThreadNew("Reader", Reader, 0);
    RunAllThread();
}

void Writer() {
    for (int i = 0; i < 40; i++) {
        char c = PrepareRandomChar();
        buffer[i%8] = c;
    }
}

void Reader() {
    for (int i = 0; i < 40; i++) {
        char c = buffer[i%8];
        ProcessChar(c);
    }
}
```

以上代码有许多问题，总结如下：

* 如果 Reader 的读速度高于 Writer 的写速度，Reader 就会读取尚未被 Writer 写入的 buff，导致 Reader 读到无用信息
* 如果 Writer 的写速度高于 Reader 的读速度，Writer 就会写入尚未被 Reader 读取的 buff，导致 Writer 覆盖尚未读取的信息

因此我们需要保证：

* Writer 不会写入尚未被读取的 buff
* Reader 不会读取尚未被写入的 buff

```c
char buffer[8];
Semaphore emptyBuffers(8);
Semaphore fullBuffers(0);

int main() {
    InitThreadPackage(false);
    ThreadNew("Writer", Writer, 0);
    ThreadNew("Reader", Reader, 0);
    RunAllThread();
}

void Writer() {
    for (int i = 0; i < 40; i++) {
        char c = PrepareRandomChar();
        SemaphoreWait(emptyBuffer);
        buffer[i%8] = c;
        SemaphoreSignmal(fullBuffers);
    }
}

void Reader() {
    for (int i = 0; i < 40; i++) {
        SemaphoreWait(fullBuffers);
        char c = buffer[i%8];
        SemaphoreSignal(emptyBuffers);
        ProcessChar(c);
    }
}
```

emptyBuffers 初始值为 8，因此 Writer 在没有 Reader 读取的情况下可以连续写入 8 个字符；fullBuffers 的初始值为 0，因此 Reader 在没有 Writer 写入的情况下无法从 buffer 读取任何字符。每当 Writer 发现还有尚未写入或者虽然写入但已经被读取的空间，它就可以继续写入，写完后 Writer 立刻提醒 Reader 多了一个新写入的字符；每当 Reader 发现还有尚未读取但已经被写入的空间，它就可以继续读取，读完后 Reader 立刻提醒 Writer 多了一个可写入的字符空间。

再看看一下几种设定：

```c
Semaphore emptyBuffers(4);
Semaphore fullBuffers(0);
```

该设定并不会影响代码的正确性，只是 Writer 和 Reader 的读写窗口从 8 减少到了 4，没有完全利用大小为 8 的 buffer 空间。同理，我们也可以将窗口减少到 1，这样会发生 Writer 和 Reader 交替工作的情况。窗口为 8 使得读写吞吐量 \(throughput\) 达到最大。

但如果把窗口减少到 0，则Reader 和 Writer 都在等待对方而产生死锁。

```c
Semaphore emptyBuffers(7);
Semaphore fullBuffers(1);
```

如果是以上这种设定，则可能出现 Reader 读取到 Writer 还没有写入的 buffer 区域。

```c
Semaphore emptyBuffers(16);
Semaphore fullBuffers(0);
```

如果是以上这种设定，则可能出现 Writer 将自己之前写入的尚未被 Reader 读取的 buffer 区域。

### Dining Philosophers Problem

\(图1\)

如上图所示，有五个哲学家在一个餐桌上吃饭，他们必须同时拿到左右两个叉子才能开始吃饭，模拟代码如下：

```c
Semaphore forks[] = {1, 1, 1, 1, 1};

void Philosopher(int id) {
    for (int i = 0; i < 3; i++) {
        Think();
        SemaphoreWait(forks[id]);
        SemaphoreWait(forks[id+1]);
        Eat();
        SemaphoreSignal(forks[id]);
        SemaphoreSignal(forks[id+1]);
    }
    Think();
}
```

但这种方式可能出现死锁，循环依赖 1 -&gt; 2 -&gt; 3 -&gt; 4 -&gt; 5 -&gt; 1，如此一来，哲学家们将永远无法吃饭。解决这个问题有很多方式，其中最粗鲁的一种就是将哲学家取左右叉的代码都放到 critical region 中；另外一种比较轻巧的方法是只允许四个哲学家开始拿叉子：

```c
Semaphore forks[] = {1, 1, 1, 1, 1};
Semaphore numAllowedToEat(4);

void Philosopher(int id) {
    for (int i = 0; i < 3; i++) {
        Think();
        SemaphoreWait(numAllowedToEat);
        SemaphoreWait(forks[id]);
        SemaphoreWait(forks[id+1]);
        Eat();
        SemaphoreSignal(forks[id]);
        SemaphoreSignal(forks[id+1]);
        SemaphoreSignal(numAllowedToEat);
    }
    Think();
}
```

#### 参考

* [Stanford-CS107-lecture-16](https://www.youtube.com/watch?v=OGHN_zVTMMo&t=0s&index=16&list=PL9D558D49CA734A02)



