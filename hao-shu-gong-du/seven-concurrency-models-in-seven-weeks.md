# Seven Concurrency Models in Seven Weeks

### Day 1: Mutual Exclusion and Memory Models

#### Take-aways

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

