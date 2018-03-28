# 第十八课 - 复杂多线程问题

## 卖冰淇淋

这个系统里面有四个角色，1 位收银员 \(Cashier\)、10 位顾客 \(Customers\)、10 - 40 位员工 \(Clerks\) 和 1 位经理 \(Manager\)，如下图所示：

（图1）

角色之间的关系如下：

* 每位顾客可能向员工购买 1 - 4 个冰淇淋，且顾客排队购买，存在有先后顺序。
* 每位员工只能同时制作 1 个冰淇淋。
* 每位员工完成冰淇淋制作后，必须经过经理检查。经理同意可以将冰淇淋交给顾客；经理不同意则需要重新制作。
* 经理在同一时间只能检查 1 个冰淇淋。
* 每位顾客拿到自己所购买的所有冰淇淋后，按排队顺序与收银员结账，收银员只能同时为一名顾客结账。

#### 构建程序主体

```c
int main() {
    int totalCones;

    InitThreadPackage();
    SetupSemaphores();
    for (int i = 0; i < 10; i++) {
        int numCones = RandomInteger(1, 4);
        // Clerk Thread 由 Customer Thread 来 spawn
        ThreadNew("", Customer, 1, numCones);
        totalCones += numCones;
    }

    ThreadNew("", Cashier, 0);
    ThreadNew("", Manager, 1, totalCones);
    RunAllThreads();
    FreeSemaphores();
    return 0;
}
```

#### Manager Thread

```c
struct inspection {
    bool passed;         // init to false
    Semaphore requested; // init to 0
    Semaphore finished;  // init to 0
    Semaphore lock;      // init to 1 , get the access to manager's office
}

void Manager(int totalConesNeeded) {
    int numApproved = 0;
    int numInspected = 0;

    while (numApproved < totalConesNeeded) {
        SemaphoreWait(inspection.requested);
        numInspected++;
        inspection.passed = RandomChoice(0, 1);
        if (inspection.passed) {
            numApproved++;
        }
        SemaphoreSignal(inspection.finished);
    }
}
```

#### Clerk Thread

```c
void Clerk(Semaphore semaToSignal) {
    bool passed = false;
    while(!passed) {
        MakeCone();
        SemaphoreWait(inspection.lock);
        SemaphoreSignal(inspection.requested);
        SemaphoreWait(inspection.finished);
        passed = inspection.passed;
        SemaphoreSignal(inspection.lock);
        SemaphoreSignal(semaToSignal);
    }
}
```

#### Customer Thread

```c
void Customer (int numCones) {
    Browse();
    Semaphore clerksDone; // init to 0

    for (int i = 0; i < numCones; i++) {
        ThreadNew("...", Clerk, 1, clerksDone);
    }

    for (int i = 0; i < numCones; i++) {
        SemaphoreWait(clerksDone);
    }
    SemaphoreFree(clerksDone);

    WalkToCashier();

    SemaphoreWait(line.lock);
    int place = line.number++;
    SemaphoreSignal(line.lock);
    SemaphoreSignal(line.requested);
    SemaphoreWait(line.customers[place]);
}
```

#### Cashier Thread

```c
struct line {
    int number;               // init to 0
    Semaphore requested;      // init to 0
    Semaphore customers[10];  // init to [0, ..., 0]
    Semaphore lock;           // init to 1
}

void Cashier () {
   for (int i = 0; i < 10; i++) {
       SemaphoreWait(line.requested);
       checkout(i);
       SemaphoreSignal(line.customers[i]);
   } 
}
```

#### 参考

* [Stanford-CS107-lecture-18](https://www.youtube.com/watch?v=ynwh5O3jVRM)



