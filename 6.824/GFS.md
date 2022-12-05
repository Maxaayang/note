![[GFS1.excalidraw]]
故障随时都会发生，要让机器自己去解决，而不是人工解决

TOLERNCE: 复制 2-3 个副本
对于单个服务器来说，容错能力是很差的

![[GFS2.excalidraw]]

![[GFS3.excalidraw]]
GFS适用于大型数据的连续访问

![[GFS4.excalidraw]]

只有primary在有资格和master进行过期时间判断

为了能够去重启master，并保证不会丢失文件系统中的任何信息，master会将所有的信息都保存在磁盘上，不仅仅是保存在内存中，读操作是在内存中进行的，写操作必须在磁盘中进行，实际中，master会将所有的操作记录以日志的形式放在磁盘中

primary存放在内存中，是因为master重启之后就会忘记哪一个是primary，它可以等待60秒过期时间，然后master就会知道没有任何的primary可用于该chunk

使用LOG而不使用数据库的原因：在磁盘上有些是用B树或者哈希表进行存储的，这样在对日志进行追加操作的时候就会非常高效，因为当磁盘机械臂旋转一次到该日志文件末尾的时候，可以将近期需要添加的日志记录一次性最佳到日志文件末尾，如果使用数据库的话，想要表达出数据的真实结果，如果是B树，那么就必须在磁盘上随机找个地方并写入一点数据(因为数据库存储时并不考虑数据存储顺序，要符合真实的B树就要通过上个数据节点的page id去做数据追加)，所以LOG可以更快的在磁盘上实现这种数据形式的存储。


## READ
![[GFS5.excalidraw]]

## WRITE
![[GFS6.excalidraw]]


### NO PRIMARY
master需要找到包含了最新chunk副本数据的服务器集合

==为什么使用版本号==
master可以根据它整理出哪些Chunk服务器保函最新的Chunk

过期时间 lease可以确保最终不会有两个primary，防止一个primary挂掉，没有过期机制，在另一个primary上位之后，挂掉的那只重启后起冲突

所有的Chunk副本都相同，并且所有的Secondary的版本号都相同，只有当master指定了一个新的primary时，版本号才会改变。
版本号只有在master不知道primary是谁的情况下才会增加。

如何解决为同一个chunk指定了两个primary的问题？
使用lease，在指定时间之前给予primary的权利

为什么要指定一个新的有问题的primary？
因为clients总是先请求master

![[GFS7.excalidraw]]




















