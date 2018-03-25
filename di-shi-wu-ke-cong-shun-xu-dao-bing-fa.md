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

#### web page loading

浏览器载入网页的过程是并行、并发大显身手的地方。假设没有它们，浏览器只能使用一个线程去连接服务器，获取网页 html、css以及图片等其它的静态文件

save time in network connection

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

Introduce semaphore

SemaphoreWait\(lock\);

SemaphoreSignal\(lock\);

support atomic --

value is not allowed to get negative number, block and pulls itself from process

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
        printf("sell a tiket");
    }
    SemaphoreSignal(lock);
}
```



