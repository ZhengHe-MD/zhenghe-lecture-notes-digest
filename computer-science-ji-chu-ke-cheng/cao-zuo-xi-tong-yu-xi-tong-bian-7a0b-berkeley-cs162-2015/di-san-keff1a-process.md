# 第三课：进程 \(Process\)

### Kernel Mode Transfers

以 web 服务为例，讨论一下进程在 user 与 kernel 之间转化的过程：

（图1）

1. server 的用户进程调用 system call --- network socket read，进入 kernel，监听请求
2. 请求到达后，进入缓存，发起 interrupt
3. kernel 将数据复制到 server 的用户进程中



