# 第十五课 - 从顺序 \(sequential\) 到并发 \(concurrent\)

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



