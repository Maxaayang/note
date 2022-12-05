Replication的两种方式
- state transfer
- replicated state machine

state transfer：如果一个服务器有两个replica，replicas要做的就是与主服务器保持同步，如果primary挂了，backup就可以接管它所需要的一切

工作原理：primary发送一个完整的状态副本给backup



replicated state machine: 复制某些内部操作

操作的大小是远小于状态的大小的