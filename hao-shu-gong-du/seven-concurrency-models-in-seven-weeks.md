# Seven Concurrency Models in Seven Weeks

### Day 1: Mutual Exclusion and Memory Models

#### take-aways

* Synchronize all access to shared variables
* Both the writing and the reading threads need to use synchronization
* Acquire multiple locks in a fixed, global order
* Don't call alien methods while holding a lock
* Hold locks for the shortest possible amount of time

#### References

[William Pugh: The Java Memory Model](http://www.cs.umd.edu/~pugh/java/memoryModel/)

[JavaWorld: Double-checked locking: Clever, but broken](https://www.javaworld.com/article/2074979/java-concurrency/double-checked-locking--clever--but-broken.html)

[JSR-133: Java Memory Model and Thread Specification](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)

[Safe Publication and Safe Initialization in Java](https://shipilev.net/blog/2014/safe-public-construction/)

[Github: Seven Concurrency Models in Seven Weeks](https://github.com/ZhengHe-MD/seven-concurrency-models-in-seven-weeks)

### Day 2: Beyond Intrinsic Locks

#### take-aways

* Intrinsic locks are convenient but limited
  * no way to interrupt a thread that's blocked as a result of trying to acquire an intrinsic lock
  * no way to time out while trying to acquire an intrinsic lock
  * exactly one way to acquire an intrinsic lock

* ReentrantLock
  * Interruptible locking
  * Timeouts
  * Hand-over-Hand locking
  * Condition Variables
* Atomic Variables: the foundation of non-blocking, lock-free algorithms

#### self-study

* ReentrantLock supports a fairness parameter. What does it mean for a lock to be "fair"? Why might you choose to use a fair lock? Why might you want?
  Fairness means all threads are fairly granted a change to execute without starvation. When several threads are waiting on one lock to be released, you want the thread waiting longer get a better chance enter critical area, you might choose to use a fair lock.

#### references

[Starvation and Fairness](http://tutorials.jenkov.com/java-concurrency/starvation-and-fairness.html)

[Oracle Java Docs: ReentrantLock](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/ReentrantLock.html)



