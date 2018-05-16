# 第三课：进程 \(Process\)

从第二课的讨论中，我们已经了解：

* 在 user processes 与 kernel 之间切换
* kernel 可以切换不同的用户进程，实现并发
* 保护 user processes 之间，以及 user processes 与 kernel 之间互不干扰

但你可能会有很多疑问：

* OS 如何表示进程？
* OS 如何选取下一个运行的进程？
* OS 在切换进程时，如何记录进程的运行时信息？
* kernel 的 stack、heap 是怎样的？
* 大量切换进程，同时记录它们的运行时信息，会不会浪费内存？

本节我们就将进入这些细节中了解 OS 中的进程。

#### Process Control Block \(PCB\)

在 Kernel 中用于表示用户进程的数据结构叫作 PCB。PCB 包含进程的重要信息，如：

* 进程状态 \(running, ready, blocked, ...\)
* 寄存器状态
* Process Id \(PID\)、用户、优先级……
* 执行时间……
* 内存空间、地址转化信息……

#### Kernel Scheduler

Kernel Scheduler 将这些 PCB 组织起来，利用一些 scheduling algorithms 来选择下一个运行的进程，这个过程可以用如下代码概括：

```c
while (true) {
    if (readyProcesses(PCBs)) {
        nextPCB = selectProcess(PCBs);
        run(nextPCB);
    } else {
        run_idle_process();
    }
}
```

代码中的 selectProcess，就隐藏着五花八门的调度策略。但不论是哪种调度策略，都需要考虑公平与效率，这在现实生活中也是如此。

#### Safe Kernel Mode Transfers







