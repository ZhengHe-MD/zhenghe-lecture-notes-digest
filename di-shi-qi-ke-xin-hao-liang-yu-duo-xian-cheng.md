# 第十七课 - 信号量与多线程（2）

### 多文件下载

要解决的问题如下：

* 同时下载多个文件
* 计算出下载的总字节数

我们可能第一反应是这样：

```c
int DownloadSingleFile(const char *server, const char *path);

int DownloadAllFiles(const char *server, 
                     const char *files[],
                     int n) {
    int totalBytes = 0;
    Semaphore lock = 1;
    for (int i = 0; i < n; i++) {
        ThreadNew("...", DownloadHelper, 4
            server, files[i], &totalBytes, lock);
    }
    return totalBytes;
}

void DownloadHelper(const char *server,
                    const char *path,
                    int *numBytesp,
                    Semaphore lock) {
    int bytesDownloaded = DownloadSingleFile(server, path);
    SemaphoreWait(lock); // why not put SW before bytesDownloaded
    (*numBytesp) += bytesDownloaded;
    SemaphoreSignal(lock);                   
}
```

首先这里需要注意 DownloadHelper 中 SemaphoreWait\(lock\) 语句放置的位置，如果它被放在了 DownloadSingleFile 语句之前，那么并发就无法进行，第二个文件下载必须在第一个文件下载完成之后才能开始，因此在书写相关代码时，要注意将 Thread-safe 的代码放到 critical region 之外，从而得到最高的并发效率。

按照如上做法，可能导致 return totalBytes 在 DonwloadHelper 线程完成之前就执行，而返回错误的数值。我们可以利用信号量来解决这个问题：

```c
int DownloadAllFiles(const char *server, 
                     const char *files[],
                     int n) {
    Semaphore childrenDone = 0;
    int totalBytes = 0;
    Semaphore lock = 1;
    for (int i = 0; i < n; i++) {
        ThreadNew("...", DownloadHelper, 4
            server, files[i], &totalBytes, lock);
    }
    for (int i = 0; i < n; i++) {
        SemaphoreWait(childrenDone);
    }
    return totalBytes;
}

void DownloadHelper(const char *server,
                    const char *path,
                    int *numBytesp,
                    Semaphore lock,
                    Semaphore parentToSignal) {
    int bytesDownloaded = DownloadSingleFile(server, path);
    SemaphoreWait(lock); // why not put SW before bytesDownloaded
    (*numBytesp) += bytesDownloaded;
    SemaphoreSignal(lock);
    SemaphoreSignal(parentToSignal);         
}
```

注意这里 DownloadAllFiles 中的 ThreadNew 和 SemaphoreWait 不应该放在同一个 for 循环中，如果它们在同一个循环中，同样会使得并发失效。

计算 totalBytes 还有另外一种做法，初始化一个 int\[\]，大小为总文件数量，这样就可以不需要使用 Semaphore lock，在第二个 for 循环结束后，直接将 int\[\] 求和即可得到 totalBytes 的计算结果。

#### 参考

* [Stanford-CS107-lecture-17](https://www.youtube.com/watch?v=kF3eSQTFagQ&t=670s&index=17&list=PL9D558D49CA734A02)



