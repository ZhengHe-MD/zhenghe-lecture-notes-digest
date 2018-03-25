# 第十五课 - 从顺序到并发和并行

买卖车票是一个常见的场景，且通常有多个中介在售卖同一列车的车票。假设该列车共有 100 个座位、 150 张票，有 10 个中介在销售该列车的票，一种做法是给每个中介 15 张票的额度，让它们卖票，不同中介之间卖的票各不相同。这种场景的模拟代码如下文所示：

```c
int main() {
    int numAgents = 10;
    int numTickets = 150;

    for (int agent = 1; agent <= numAgents; agent++) {
        SellTickets(agent, numTickets/numAgents);
    }

    return 0;
}

void SellTickets (int agentID, int numTicketsToSell) {
    while (numTicketsToSell > 0) {
        printf("Agent %d sells a ticket \n", agentID);
        numTicketsToSell--;
    }

    printf("agent %d: All done \n", agentID);
}
```

上文中的代码并未真实反应同时售票的过程，而是中介1、中介2、中介3...、中介10 依次把各自额度中的票卖完，这并没有提高售票效率。真实场景中，所有中介理应相互独立，中介 2 没有必要等待中介 1 销售完成后才开始销售，因此利用多线程，我们再次尝试模拟该场景：

```c
int main() {
    int numAgents = 10;
    int numTickets = 150;

    InitThreadPackage(false); // false 表示不要打印 debug 信息
    for (int agent = 1; agent <= numAgents; agent++) {
        char name[32];
        sprintf(name, "Agent %d Thread", agent);
        ThreadNew(name, SellTickets, 2, agent, numTickets/numAgents); // 此时 thread 尚未执行
    }
    RunAllThreads(); // start heart beat
    return 0;
}

void SellTickets (int agentID, int numTicketsToSell) {
    while (numTicketsToSell > 0) {
        printf("Agent %d sells a ticket \n", agentID);
        // this is not a atomic operation
        // c instructions are not atomic
        numTicketsToSell--;
        // simulate the real world
        if (RandomChange(0.1)) ThreadSleep(1000);
    }

    printf("agent %d: All done \n", agentID);
}


// Agent 1 sells a ticket.
// ...
// ...
// Agent 2 sells a ticket
// Agent 3 sells a ticket
// ...
// ...
// Agent 7 All done.
// Agent 8 sells a ticket
// Agent 8 All done.
// ...
// ...
// Agent 4 All done
```

上文的 main 中，我们通过 InitThreadPackage、ThreadNew 和 RunAllThreads 来模拟 10 个中介同时卖票的过程。在单核 CPU 的环境下，这种同时卖票的过程是通过在不同的中介之间快速切换来实现的：当 RunAllThreads 运行时，Thread Manager 将 CPU 的计算资源分成时间段，然后将每个时间段分配给不同的 Thread，由于只有一个核，一组寄存器，因此这些 Threads，即 10 个中介，只能轮流进入核中来推进售票流程，这就是所谓的并发；在多核 CPU 环境下，这时候可能有多个 Thread 同时在运行，即同时可能有多个中介进入不同的核中来推进售票流程，这就是所谓的并行。并发与并行过程对应用开发者不可见。无论是并发还是并行，在一些实际场景中都能起到提升效率的效果，举例如下：

##### web page loading

浏览器载入网页的过程是并行、并发大显身手的地方。假设没有它们，浏览器只能使用一个线程去连接服务器，获取网页 html、css以及图片等其它的静态文件，这些文件可能各自来自不同的服务器，而与这些服务器分别建立连接的过程本身所需网络等待时间可能超过实际建立连接之后下载数据所需的时间，因此这个过程比较低效。但如果浏览器能使用多个线程去连接服务器，那么它就能够在线程 A 等待网络连接的过程中切换到线程 B ，等线程 A 的连接响应时，再从线程 B 切换回线程 A，这时候页面的整体载入时间就能够大大缩短。

回到车票销售的场景，引入并发、并行确实能够提高资源利用率，但与此同时也引入了线程之间的信息不对称的问题，如下文代码所示所示，中介不再销售固定数量的不同的车票，而是通过访问余票总数来判断是否可以继续售票。如果线程 A 执行完  while \(\*numTickets &gt; 0\) 的判断后，被 Thread Manager 剥夺 CPU 时间， 同时 Thraed Manager 会把寄存器中的所有信息保存到 A 中再切换到线程 B，这时 B 看到了与 A 相同的余票值，作出同样的判断。这样就出现了信息不对称的问题。

```c
int main() {
    int numAgents = 10;
    int numTickets = 150;

    InitThreadPackage(false);
    for (int agent = 1; agent <= numAgents; agent++) {
        char name[32];
        sprintf(name, "Agent %d Thread", agent);
        ThreadNew(name, SellTickets, 2, agent, &numTickets);
    }
    RunAllThreads(); // start heart beat
    return 0;
}



void SellTickets(int agent, int *numTicketsp) {
    while (*numTicketsp > 0) {
        // get swapped out after testing
        (*numTicketsp)--;
    }
}
```

为解决不同线程间的信息不对称问题，我们引入信号量 \(Semaphore\) 的概念。信号量可以理解成一个可以完成原子 \(atomic\) 自减、自增操作的整数，在本课的设定中，它的值永远不能小于 0 。SemaphoreWait 函数会在 semaphore 取值大于 0 时完成原子自减，在 semaphore 取值为 0 时让线程进入等待状态；SemaphoreSignal 函数会将 semaphore 的值原子自增。我们可以将 SemaphoreWait 简单地理解成等待上公厕，当公厕里有人 \(semaphore 取值等于 0 \) 时等待，当公厕里没人 \(semaphore 取值大于 0 \) 时进入并锁上门的过程；将 SemaphoreSignal 理解成上完厕所后打开门的过程，这时候我们可以将上文中的代码改进如下：

```c
int main() {
    int numAgents = 10;
    int numTickets = 150;
    Semaphore lock = SemaphoreNew(-, 1);
    InitThreadPackage(false);
    for (int agent = 1; agent <= numAgents; agent++) {
        char name[32];
        sprintf(name, "Agent %d Thread", agent);
        ThreadNew(name, SellTickets, 3, agent, &numTickets, lock);
    }
    RunAllThreads(); // start heart beat
    return 0;
}


void SellTickets(int agent, int *numTicketsp, Semaphore lock) {
    while (true) {
        SemaphoreWait(lock);
        if (*numTicketsp == 0) break;
        (*numTickets)--;
        SemaphoreSignal(lock);
        printf("sell a ticket");
    }
    SemaphoreSignal(lock);
}
```

我们称 SemaphoreWait 与 SemaphoreSignal 之间的区域为 critical area，这个区域只能有一个线程进去。值得一提的是，critical area 应当放置尽可能少的代码，不影响线程间通信的代码尽量放在 critical area 之外，如上面的 printf\("sell a ticket"\)

#### 参考

* [Stanford-CS107-lecture-15](https://www.youtube.com/watch?v=omE3YYpHhLo&list=PL9D558D49CA734A02&index=15\)



