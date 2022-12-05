**容错性**
- availability：建立在特定的错误之上
- recoverability
	- NV storage：非易失性存储更新起来很昂贵，所以构建高性能容错系统非常繁琐，所以要尽量避免写入非易失性存储。
	- Replication：任何两个副本总会意外的偏离同步状态，不再正确。

一致性（consistency）
强一致性：PUT一定是一个完整的PUT
弱一致性：不保证任何类似GET的操作，可以看到最新的PUT写入的值
强一致性虽然可以保证看到的是最新的值，但是实现却是很昂贵的，因为这意味着要做很多的通信来保证强一致性的实现。


# MapReduce

![MapReduce笔记](https://superlova.github.io/2021/05/04/%E3%80%90%E8%AE%BA%E6%96%87%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0%E3%80%91MapReduce-Simplified-Data-Processing-on-Large-Clusters/#3-2-Master%E8%8A%82%E7%82%B9%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)

![Avu9aP](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/Avu9aP.png)

list(k2, v2)是map把 (k1, v1)分布式计算后的中间结果集合
reduce(k2, list(v2)) 是 reduce 函数根据 k2 的值来合并 v2，最终想要的结果是 list(v2)

## MapReduce 执行流程
![8cqfip](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/8cqfip.png)
1.  在map阶段，MapReduce会对要处理的数据进行分片（split）操作，为每一个分片分配一个MapTask任务。将输入分成M部分，每部分的大小一般在16M~64M之间（用户来定义）。输出也分为R部分（？）。然后在各个机器上fork程序副本。
    
2.  选定集群中的一个机器为master节点，负责分发任务；其他节点是worker，负责计算和向master提交任务结果。
    
3.  之前指定了M个map任务和R个reduce任务，master节点给每个空闲的worker分配一个map任务或者一个reduce任务。
    
4.  被分配map任务的worker会读取对应的输入片段，输入用户定义的map函数，输出中间结果，将这些中间结果缓存在内存中。这些中间结果会定期地保存在本地此版中。由partition函数将其分成R部分。worker负责将这些缓存数据对在磁盘中的位置上传给master。
    
5.  master负责收集map worker发送回来的数据对位置，然后把这些位置发送给 reduce worker。当一个reduce worker把这些中间结果读取完毕后，它会首先对这些中间结果排序，这样同样key的中间结果就会相邻了。
    

key很多种类的情况下，排序是有必要的吗？实践表明，排序是有必要的，因为数据分片后，往往同一key的数据在同一片M中。这体现了数据在空间上的局部性。

但是如果数据量过大，中间结果过多，我们可能需要外部排序。

1.  reduce worker迭代所有中间结果，由于这些中间信息按照key排序过了，因此很容易获得同样key的所有键值对集合(key, {values})。将这一部分整合key后的信息传递给Reduce函数。Reduce函数的输出被追加到所属分区R的输出文件中。
    
2.  当所有Map和Reduce任务都完成后，master唤醒用户程序，用户程序返回调用结果。

## Master 节点的数据结构

Master节点会保存每个map任务和reduce任务的执行状态（空闲 idle、正在执行 in-progress，执行完毕 completed），还会给被分配任务了的worker保存一个标注。

Master还会像一个管道一样储存map任务完成后的中间信息存储位置，并把这些位置信息传输给reduce worker。

## 容错机制
### Worker Failure

master会周期性地ping每个worker，规定时间内没有返回信息，则master将其标记为fail。master将所有由这个失效的worker做完的（completed）、正在做的（in-progress）`map`任务标记为初始状态（idle），等待其他worker认领任务。

worker故障时，对于map任务结果和reduce任务结果的处理方法有所不同。map任务的结果由于需要通知master存储位置，中途中断会导致存储位置丢失，因此失败的map任务需要重新执行；reduce任务的结果存储位置在全局文件系统上，因此不需要再次执行。

当worker B接手了失败的worker A的task，所有reduce worker都会知晓。因此所有还没有从worker A哪里获得数据的reduce worker会自动向worker B获取数据。

### Master Failure

首先要有检查点（checkpoint）机制，周期性地将master节点存储的数据保存至磁盘。

但是由于只有一个master进程，因此master失效后MapReduce运算会中止。由用户亲自检查，根据需要重新执行MapReduce。

### Semantics in the Presence of Failures

对于一个确定性的程序而言，程序的寓意就是程序的执行顺序；但是对于非确定性程序而言，也就是执行多次可能得到不同结果的程序而言，语义是否能保证，直接与最后结果是否正确相关。

MapReduce保证，当Map和Reduce的操作都是确定性函数（只要相同输入就会得到相同输出的函数），那么MapReduce处理得到的结果也都是确定性的，不论集群内部有没有错误、执行顺序。

这种强保证是由Map和Reduce中的commit操作的原子性来保证的。

每个 in-progress task 都将其输出写进私有临时文件中。每个reduce产生一个私有临时文件，每个map产生R个私有临时文件（因为对应R个reduce任务）。

当map任务完成，map worker发送给master的是那R个临时文件的名称，并标注“我做完了”。master在收到消息后，就将这R个文件名记录在自己的数据结构中。**如果这个时候由于某些错误**，master又收到一遍“我做完了”，master将会忽略。

当reduce任务完成，reduce worker把临时文件重命名为最终的输出文件名。重命名操作是原子的，即要不就全部重命名成功，要不就一个都不会重命名。这里存在一种语义风险，那就是如果同一个reduce task在多台机器上执行，同一个结果文件有可能被重命名多次。为保证最终文件系统只包含一个reduce任务产生的数据，MapReduce依赖底层文件系统提供的重命名操作（？）。

### Locality

这一部分的设计要尽可能节约带宽，因为带宽是相当匮乏的资源。

MapReduce的解决方案是通过GFS (Google File System)将每个文件分成 64 MB的块，然后将每块保存几个副本（通常为3份）在不同的机器上。MapReduce 尽量将这些位置信息保存下来然后尽量将含有某个文件主机的任务分配给它，这样就可以减少网络的传递使用。如果失败，那么将会尝试从靠近输入数据的一个副本主机去启动这个任务。当在一个集群上执行大型的 MapReduce 操作的时候，输入数据一般都是本地读取，减少网络带宽的使用。


## Task Granularity

理想状况下，M和R应当与worker数目大很多，这样才能提高集群的动态负载均衡能力，并且能加快故障恢复的速度，原因是失效的worker上执行的map任务可以分布到所有其他的worker机器上执行。

但是M和R也是有限制的，这一部分限制主要是由于master需要执行O(M+R)次调度。

我们通常会按照这样的比例执行：M=200,000，R=5,000，worker有2,000台。

## Backup Tasks

长尾分布现象（或者说“水桶效应”）在MapReduce中也有体现，因为MapReduce计算时间往往取决于其运行速度最慢的worker。

有一个办法来减少“straggler”（落伍的人），master会在任务快完成时，调用backup进程来解决那些 in-progress 任务。这样，无论是原来的进程还是 backup 进程中的哪个先完成，master都立即将其标记为完成。



