
---
annotation-target: note/22-distributed_note.pdf
annotation-target-type: pdf
---


>%%
>```annotation-json
>{"created":"2022-05-26T03:08:40.063Z","text":"111","updated":"2022-05-26T03:08:40.063Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":509,"end":650},{"type":"TextQuoteSelector","exact":"An important goal in designing a distributed DBMS is fault tolerance (i.e., avoiding a singleone node failure taking down the entire system).","prefix":"on in a distributedenvironment. ","suffix":"Differences between parallel and"}]}]}
>```
>%%
>*%%HIGHLIGHT%%An important goal in designing a distributed DBMS is fault tolerance (i.e., avoiding a singleone node failure taking down the entire system).*
>%%LINK%%[[#^4wwbzzgo8co|show annotation]]
>%%COMMENT%%
>分布式的一个重要的涉及目标就是对错误的容忍
>%%TAGS%%
>
^4wwbzzgo8co

# System Architectures

![m3B4rd](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/m3B4rd.png)

>%%
>```annotation-json
>{"created":"2022-05-26T03:15:44.003Z","updated":"2022-05-26T03:15:44.003Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":1507,"end":1578},{"type":"TextQuoteSelector","exact":"A single-node DBMS uses what is called a shared everything architecture","prefix":"d store objects in the database.","suffix":". This single node executes work"}]}]}
>```
>%%
>*%%HIGHLIGHT%%A single-node DBMS uses what is called a shared everything architecture*
>%%LINK%%[[#^94y511ze8eb|show annotation]]
>%%COMMENT%%
>所有的非分布式的DBMS都使用这种方案
>%%TAGS%%
>
^94y511ze8eb


## Shared Memory

>%%
>```annotation-json
>{"created":"2022-05-26T03:17:13.377Z","updated":"2022-05-26T03:17:13.377Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":1695,"end":1886},{"type":"TextQuoteSelector","exact":"An alternative to shared everything architecture in distributed systems is shared memory. CPUs have accessto common memory address space via a fast interconnect. CPUs also share the same disk","prefix":"ess space and disk.Shared Memory","suffix":".In practice, most DBMSs do not "}]}]}
>```
>%%
>*%%HIGHLIGHT%%An alternative to shared everything architecture in distributed systems is shared memory. CPUs have accessto common memory address space via a fast interconnect. CPUs also share the same disk*
>%%LINK%%[[#^qbz6mmg6uo|show annotation]]
>%%COMMENT%%
>在分布式环境下的一种选择方案，常见于高性能计算领域。如果这些worker彼此之间想相互通信，它们可以做那些 shared-everything 系统下所做的事，可以往一个全局数据结构中写入数据，或者通过IPC发送一条消息，运行在其他机器上的 worker 会看到这些东西。
>在这种 shared-everything 系统下，当我们使用两阶段锁的时候，如果我们想告诉另一个worker我拿着这个 tuple 对应的锁，就会往内存中的 lock table 中添加一个条目。
>如果一个 worker 想去获取某个 tuple 对应的锁，它只需去更新下这个全局 lock table，接着 Messaging Fabric 是用来保证确保所有 worker 的接触的所有东西都是一致的
>%%TAGS%%
>
^qbz6mmg6uo


>%%
>```annotation-json
>{"created":"2022-05-26T03:23:12.204Z","updated":"2022-05-26T03:23:12.204Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":1900,"end":2123},{"type":"TextQuoteSelector","exact":"most DBMSs do not use this architecture, as it is provided at the OS / kernel level. It also causesproblems, since each process’s scope of memory is the same memory address space, which can be modifiedby multiple processes.","prefix":"hare the same disk.In practice, ","suffix":"Each processor has a global view"}]}]}
>```
>%%
>*%%HIGHLIGHT%%most DBMSs do not use this architecture, as it is provided at the OS / kernel level. It also causes problems, since each process’s scope of memory is the same memory address space, which can be modifiedby multiple processes.*
>%%LINK%%[[#^td3ka3jj64|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^td3ka3jj64


>%%
>```annotation-json
>{"created":"2022-05-26T03:24:47.851Z","updated":"2022-05-26T03:24:47.851Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":2123,"end":2267},{"type":"TextQuoteSelector","exact":"Each processor has a global view of all the in-memory data structures. Each DBMS instance on a processorhas to “know” about the other instances.","prefix":"e modifiedby multiple processes.","suffix":"Fall 2021 – Lecture #22 Introduc"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Each processor has a global view of all the in-memory data structures. Each DBMS instance on a processorhas to “know” about the other instances.*
>%%LINK%%[[#^k09mvh1pbx|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^k09mvh1pbx


## Shared Disk

云原生数据库系统都是在这种环境下运行的，因为可以获得的一个优势是，可以分别对计算资源和磁盘资源进行扩展，因为计算资源都是无状态的，数据库状态则是在它之下的，所以如果这些计算资源崩溃消失了，假设做好了日志记录，那么所有数据依然都还在磁盘中，那么就可以将这些东西放到另一个实例中，并从它们故障的地方开始继续工作。

在 shared-nothing 环境下是不容易做到的，因为每个节点都是有状态的。

>[!FAQ]
>共享内存架构与多处理器共享系统的区别：
>它们是一样的，它们是分布式DBMS，它们拥有一个跨多台物理机器的统一内存，So每台机器都有自己的主板以及它可以进行读写的物理内存，但有一层，它表示所有处理器都会觉得它们有一块很大的内存。

>[!FAQ]
在这种共享磁盘架构下，如果数据库厂商使用的是这种共享磁盘架构，是否意味着他们必须讲该数据库的所有数据放在一个地方？
No，之后会讲。

>[!example] Shared Disk Example
>![2Fdfpb](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/2Fdfpb.png)
>在磁盘的物理位置和逻辑位置之间存在一个抽象层，这些节点并不清楚存放在磁盘中的什么位置，So，它表示这里有一个我可以进行读写的文件，就像和我在本地磁盘上对文件进行操作一样。
>
>应用程序表示会跑到某个节点处拿到某个记录，可能会在此前面维护一些信息，这些信息会告诉我们去哪里获取我需要的东西，我们会去获取这个东西。
>
>在Node节点中，除了 buffer pool 中的数据以外，它里面并没有保存数据库的状态，我们不能把它里面的数据视作是持久化的，它是临时的。

>[!faq]
>因为数据存储使用了分区或分片，我们要去哪个目的地，得明确区分出那个分片或分区拥有哪些数据才行
>依然可以在计算节点层面进行分区

>[!note]
>当在某个节点进行更新的时候，该节点会发送消息给其他节点，它得自顶到底将更新信息推动到各个节点的。
>
>一种被用方案就是去推送通知告诉其他节点，并让其他节点都更新并刷新一下。



>%%
>```annotation-json
>{"created":"2022-05-26T03:26:39.322Z","updated":"2022-05-26T03:26:39.322Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":2548,"end":2670},{"type":"TextQuoteSelector","exact":"all CPUs can read and write to a single logical disk directly via an interconnect,but each have their own private memories","prefix":"kIn a shared disk architecture, ","suffix":". This approach is more common i"}]}]}
>```
>%%
>*%%HIGHLIGHT%%all CPUs can read and write to a single logical disk directly via an interconnect,but ==each have their own private memories==*
>%%LINK%%[[#^bt73qjqtq|show annotation]]
>%%COMMENT%%
>CPU上的worker有它们自己的本地内存，但是用来维护数据库持久化状态的磁盘是一种共享设备。
>%%TAGS%%
>
^bt73qjqtq


>%%
>```annotation-json
>{"created":"2022-05-26T03:29:32.540Z","updated":"2022-05-26T03:29:32.540Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":2906,"end":3149},{"type":"TextQuoteSelector","exact":"Nodes must send messages between them to learn about other node’s current state. That is, since memory islocal, if data is modified, changes must be communicated to other CPUs in the case that piece of data is inmain memory for the other CPUs.","prefix":"tion of data in the other layer.","suffix":"Nodes have their own buffer pool"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Nodes must send messages between them to learn about other node’s current state. That is, since memory islocal, if data is modified, changes must be communicated to other CPUs in the case that piece of data is inmain memory for the other CPUs.*
>%%LINK%%[[#^p23uyjkerxm|show annotation]]
>%%COMMENT%%
>因为这个CPU无法读取另一个CPU的内存（共享的是磁盘存储，可以参考824的第一集）
>%%TAGS%%
>
^p23uyjkerxm


## Shared Nothing

>%%
>```annotation-json
>{"created":"2022-05-26T06:17:16.848Z","updated":"2022-05-26T06:17:16.848Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":3391,"end":3519},{"type":"TextQuoteSelector","exact":"In a shared nothing environment, each node has its own CPU, memory, and disk. Nodes only communicatewith each other via network.","prefix":"he case ofcrashes.Shared Nothing","suffix":"It is more difficult to increase"}]}]}
>```
>%%
>*%%HIGHLIGHT%%In a shared nothing environment, each node has its own CPU, memory, and disk. Nodes only communicatewith each other via network.*
>%%LINK%%[[#^c30mphx0ei|show annotation]]
>%%COMMENT%%
>唯一需要去协调这些节点的方式是，当执行查询时，这些节点之间会直接进行通信，So，如果想去获取数据，如果查询要去访问另一台机器上的数据，就没法跑到共享磁盘处获取这个数据，因为这个架构中没有共享磁盘，也不能去读取其他机器内存中的数据，因为没法这么做，会发送一条消息去获得想要的东西。
>%%TAGS%%
>
^c30mphx0ei

==disadvantage==

>%%
>```annotation-json
>{"created":"2022-05-26T07:13:33.959Z","updated":"2022-05-26T07:13:33.959Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":3519,"end":3639},{"type":"TextQuoteSelector","exact":"It is more difficult to increase capacity in this architecture because the DBMS has to physically move datato new nodes.","prefix":"catewith each other via network.","suffix":" It is also difficult to ensure "}]}]}
>```
>%%
>*%%HIGHLIGHT%%It is more difficult to increase capacity in this architecture because the DBMS has to physically move data to new nodes.*
>%%LINK%%[[#^5exqkd4ui7x|show annotation]]
>%%COMMENT%%
>**扩容并确保一致性是这个架构方案最难的地方**
>因为需要能够运行系统，并以某种方式移动数据，以避免丢失数据，当执行查询时，就不会遇上假阴性或假阳性的问题，否则就会关闭整个系统，然后移动数据进行扩容，但并不想这么做，因为想让它始终能在线提供服务。

>%%TAGS%%
>
^5exqkd4ui7x


>%%
>```annotation-json
>{"created":"2022-05-26T07:14:13.008Z","updated":"2022-05-26T07:14:13.008Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":3640,"end":3789},{"type":"TextQuoteSelector","exact":"It is also difficult to ensure consistency across all nodes in the DBMS, since the nodes mustcoordinate with each other on the state of transactions.","prefix":"ysically move datato new nodes. ","suffix":" The advantage, however, is that"}]}]}
>```
>%%
>*%%HIGHLIGHT%%It is also difficult to ensure consistency across all nodes in the DBMS, since the nodes must coordinate with each other on the state of transactions.*
>%%LINK%%[[#^at4mh3dlp9|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^at4mh3dlp9

==advantage==


>%%
>```annotation-json
>{"created":"2022-05-26T07:14:36.805Z","updated":"2022-05-26T07:14:36.805Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":3822,"end":3958},{"type":"TextQuoteSelector","exact":"shared nothingDBMSs can potentially achieve better performance and are more efficient then other types of distributedDBMS architectures.","prefix":"The advantage, however, is that ","suffix":"3 Design IssuesDistributed DBMSs"}]}]}
>```
>%%
>*%%HIGHLIGHT%%shared nothing DBMSs can potentially achieve better performance and are more efficient then other types of distributed DBMS architectures.*
>%%LINK%%[[#^q9ydhplw3us|show annotation]]
>%%COMMENT%%
>因为现在可以只关注局部性数据，并尽可能多地通过网络移动最少的数据量。
>%%TAGS%%
>
^q9ydhplw3us

![WmW3B7](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/WmW3B7.png)



>[!note]
>如果需要添加一个新的节点，就需要从其他节点处获取该数据库的某一部分数据，这样就需要将各个节点数据都匀一点给它，接着一旦复制完了数据，就会去更新下全局状态，并将这个节点负责的范围声明。
>
>但是这很难，如果在意事务的话，不想丢失任何数据，因为不想有这么一个查询，想去访问某个数据条目时，落到了这个节点处，该节点还未被转移到新的节点，So，也许可以回应这个查询，但也许该数据已经被迁移走了，它就什么都不会返回了，虽然想要的数据在新节点中。


**使用分布式系统的优势**：
当对硬件进行垂直扩展时，收益会递减，水平扩展是添加新机器，垂直扩展是我那个一台机器上添加更多的资源，以此来让它更为强大，通常垂直扩展的成本更加昂贵。

**分布式数据库系统水平扩展和垂直扩展的优势**：
当管理一个分布式数据库系统时，通信成本更加昂贵，如果使用的是单节点的系统，速度肯定更快，因为不需要在其他不同节点之间进行协调并通过网络发送消息，但一个系统的垂直扩展是有极限的。


# Design Issues

![ftFHyy](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/ftFHyy.png)

## Homogeneous VS Heterogeneous
### Homogeneous
>%%
>```annotation-json
>{"created":"2022-05-26T08:08:01.716Z","updated":"2022-05-26T08:08:01.716Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":4988,"end":5154},{"type":"TextQuoteSelector","exact":"Every node in the cluster can perform the same set of tasks (albeit on potentiallydifferent partitions of data), lending itself well to a shared nothing architecture.","prefix":"-day systems.Homogeneous Nodes: ","suffix":" This makes provisioningand fail"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Every node in the cluster can perform the same set of tasks (albeit on potentially different partitions of data), lending itself well to a shared nothing architecture.*
>%%LINK%%[[#^w4dz2uwh43k|show annotation]]
>%%COMMENT%%
>数据库集群中的每个节点可以执行想要的任何任务，即可以将一个查询发送给任意节点，该节点会搞清楚所寻找的结果是什么，然后它们会进行一些后台任务和其他东西
>%%TAGS%%
>
^w4dz2uwh43k

==advantage==
>%%
>```annotation-json
>{"created":"2022-05-26T08:08:46.046Z","updated":"2022-05-26T08:08:46.046Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":5155,"end":5246},{"type":"TextQuoteSelector","exact":"This makes provisioningand failover “easier”. Failed tasks are assigned to available nodes.","prefix":" a shared nothing architecture. ","suffix":"Heterogeneous Nodes: Nodes are a"}]}]}
>```
>%%
>*%%HIGHLIGHT%%This makes provisioning and failover “easier”. Failed tasks are assigned to available nodes.*
>%%LINK%%[[#^2izpsgpajqw|show annotation]]
>%%COMMENT%%
>使故障预防和故障转移变得更易于处理和更好地支持，因为现在可以添加新的节点，并且可以安全地移动数据
>%%TAGS%%
>
^2izpsgpajqw

### Heterogeneous

>%%
>```annotation-json
>{"created":"2022-05-26T08:11:33.758Z","updated":"2022-05-26T08:11:33.758Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":5267,"end":5369},{"type":"TextQuoteSelector","exact":"Nodes are assigned specific tasks, so communication must happen between nodesto carry out a given task","prefix":"able nodes.Heterogeneous Nodes: ","suffix":". Can allow a single physical no"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Nodes are assigned specific tasks, so communication must happen between nodesto carry out a given task*
>%%LINK%%[[#^1y089hc2tpl|show annotation]]
>%%COMMENT%%
>可以给数据库系统中的某个节点分配指定的任务，So，如果系统运行速度变慢，想去添加新节点，需要知道应该添加的是哪种类型的节点
>%%TAGS%%
>
^1y089hc2tpl


# Partitioning Schemes


>%%
>```annotation-json
>{"created":"2022-05-26T08:18:44.318Z","updated":"2022-05-26T08:18:44.318Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":5531,"end":5640},{"type":"TextQuoteSelector","exact":"Distributed system must partition the database across multiple resources, including disks, nodes, processors.","prefix":" to other.4 Partitioning Schemes","suffix":"This process is sometimes called"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Distributed system must partition the database across multiple resources, including disks, nodes, processors.*
>%%LINK%%[[#^uz86kz8yvuh|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^uz86kz8yvuh


>%%
>```annotation-json
>{"created":"2022-05-26T08:19:02.239Z","updated":"2022-05-26T08:19:02.239Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":5795,"end":5925},{"type":"TextQuoteSelector","exact":"The DBMS may potentially send fragments of thequery plan to different nodes, then combines the results to produce a single answer.","prefix":"the query plan needs to access. ","suffix":"The goal of a partitioning schem"}]}]}
>```
>%%
>*%%HIGHLIGHT%%The DBMS may potentially send fragments of thequery plan to different nodes, then combines the results to produce a single answer.*
>%%LINK%%[[#^r4ljxoor0mj|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^r4ljxoor0mj


>%%
>```annotation-json
>{"created":"2022-05-26T08:21:57.756Z","updated":"2022-05-26T08:21:57.756Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":5925,"end":6340},{"type":"TextQuoteSelector","exact":"The goal of a partitioning scheme is to maximize single-node transactions, or transactions that only accessdata contained on one partition. This allows the DBMS to not need to coordinate the behavior of concurrenttransactions running on other nodes. On the other hand, a distributed transaction accesses data at one ormore partitions. This requires expensive, difficult coordination, discussed in the below section.","prefix":"ults to produce a single answer.","suffix":"For logically partitioned nodes,"}]}]}
>```
>%%
>*%%HIGHLIGHT%%==The goal of a partitioning scheme== is to maximize single-node transactions, or transactions that only access data contained on one partition. This allows the DBMS to not need to coordinate the behavior of concurrent transactions running on other nodes. On the other hand, a distributed transaction accesses data at one ormore partitions. This requires expensive, difficult coordination, discussed in the below section.*
>%%LINK%%[[#^9yqj8vxqrot|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^9yqj8vxqrot


>%%
>```annotation-json
>{"created":"2022-05-26T08:22:58.721Z","updated":"2022-05-26T08:22:58.721Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":6340,"end":6569},{"type":"TextQuoteSelector","exact":"For logically partitioned nodes, particular nodes are in charge of accessing specific tuples from a shareddisk. For physically partitioned nodes, each shared nothing node reads and updates tuples it contains on itsown local disk.","prefix":" discussed in the below section.","suffix":"ImplementationThe simplest way t"}]}]}
>```
>%%
>*%%HIGHLIGHT%%For ==logically partitioned nodes==, particular nodes are in charge of accessing specific tuples from a shared disk. For ==physically partitioned nodes==, each shared nothing node reads and updates tuples it contains on its own local disk.*
>%%LINK%%[[#^20p7rw8clua|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^20p7rw8clua

## Implementation

### naive data partitioning


>%%
>```annotation-json
>{"created":"2022-05-26T08:24:24.771Z","updated":"2022-05-26T08:24:24.771Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":6648,"end":6722},{"type":"TextQuoteSelector","exact":"Each node stores one table, assuming enoughstorage space for a given node.","prefix":"les is naive data partitioning. ","suffix":" This is easy to implement becau"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Each node stores one table, assuming enough storage space for a given node.*
>%%LINK%%[[#^kjco3orw1x|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^kjco3orw1x


>%%
>```annotation-json
>{"created":"2022-05-26T08:24:47.794Z","updated":"2022-05-26T08:24:47.794Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":6723,"end":6806},{"type":"TextQuoteSelector","exact":"This is easy to implement because a query is just routed to a specificpartitioning.","prefix":"storage space for a given node. ","suffix":" This can be bad, since it is no"}]}]}
>```
>%%
>*%%HIGHLIGHT%%This is easy to implement because a query is just routed to a specific partitioning.*
>%%LINK%%[[#^duw4fznkjw|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^duw4fznkjw

==disadvantage==


>%%
>```annotation-json
>{"created":"2022-05-26T08:25:17.534Z","updated":"2022-05-26T08:25:17.534Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":6807,"end":6960},{"type":"TextQuoteSelector","exact":"This can be bad, since it is not scalable. One partition’s resources can be exhausted if that onetable is queried on often, not using all nodes available","prefix":"uted to a specificpartitioning. ","suffix":". See Figure 2 for an example.Mo"}]}]}
>```
>%%
>*%%HIGHLIGHT%%This can be bad, since it is not scalable. One partition’s resources can be exhausted if that onetable is queried on often, not using all nodes available.*
>%%LINK%%[[#^jiuiguboh1|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^jiuiguboh1

### horizontal partitioning


>%%
>```annotation-json
>{"created":"2022-05-26T08:30:47.008Z","updated":"2022-05-26T08:30:47.008Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":7044,"end":7205},{"type":"TextQuoteSelector","exact":"plits a table’s tuples into disjoint subsets. Choosecolumn(s) that divides the database equally in terms of size, load, or usage, called the partitioning key(s).","prefix":"horizontal partitioning, which s","suffix":"The DBMS can partition a databas"}]}]}
>```
>%%
>*%%HIGHLIGHT%%splits a table’s tuples into disjoint subsets. Choose column(s) that divides the database equally in terms of size, load, or usage, called the partitioning key(s).*
>%%LINK%%[[#^pqfkufcjqcg|show annotation]]
>%%COMMENT%%
>逐行对表进行拆分，将一个或多个列作为 partitioning key，通过对这些 partitioning key 的值进行测试，接着会去选择将它分配给哪个分区
>%%TAGS%%
>
^pqfkufcjqcg


>%%
>```annotation-json
>{"created":"2022-05-26T08:30:54.055Z","updated":"2022-05-26T08:30:54.055Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":7205,"end":7340},{"type":"TextQuoteSelector","exact":"The DBMS can partition a database physically (shared nothing) or logically (shared disk) via hash partition-ing or range partitioning. ","prefix":" called the partitioning key(s).","suffix":"See Figure 3 for an example.Anot"}]}]}
>```
>%%
>*%%HIGHLIGHT%%The DBMS can partition a database physically (shared nothing) or logically (shared disk) via hash partition-ing or range partitioning.*
>%%LINK%%[[#^x5uhrybko2o|show annotation]]
>%%COMMENT%%
>在 shared-nothing 系统中可以使用物理分区，因为每个节点实际上会将它的分区保存在本地磁盘。
>在 shared-disk 系统中，使用的是逻辑分区，这样就无需将同一个 page 复制到多个节点上了，以此来减少需要做的协调工作量。
>%%TAGS%%
>
^x5uhrybko2o

选择使用哪个 partitioning key 是一个 np-complete 问题，因为可以做很多不同的组合。
就像是一个优化问题，通过查看 workload，可以知道该怎样让我的查询去访问这个表，会去查看这个 partition key，它等于多少，然后通过相应的算法得到想要数据的目的地。

>[!faq] 在 hash partitioning 中，根据拥有的分区数量来对 partitioning key 进行取模，就会告诉我们要去哪个分区，这样做的问题是什么？
>- 如果是用的是 hash partitioning，如果进行循序扫描，如果进行范围判断，或者等价判断，那么 hash partitioning 就是一个糟糕的想法，因为无法对一个范围进行 hash，并将这个范围中的数据都放入同一个 hash table 中。
>
>- 如果有第五个分区，就会遇上在讨论 hash table 时所遇到的相同问题，即如果重新使用5来对所有的值进行 hash，就无法保证这些数据依然在同一个分区当中，可能最终会移动整个数据库中的每一条数据，可能将在这个分区的这条数据交换并移动到另一个分区中。

### Consistent Hashing

允许在不移动任何东西的情况下，对集群中的分区进行增量更新和移除。

![B3ZWac](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/B3ZWac.png)


节点之间的空间就是从上一个分区到下一个分区之间的空间，例如 A 没了，hash 结果就落到了 B 上，B 没了，hash 结果就落到 C 上，而 hash 的结果落在 B 与 C 之间的话，那就存在 C 上面，因为是顺时针操作的。



>%%
>```annotation-json
>{"created":"2022-05-26T09:14:11.276Z","updated":"2022-05-26T09:14:11.276Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":7560,"end":7679},{"type":"TextQuoteSelector","exact":"The node that isclosest to the key in the clockwise direction is responsible for that key. See Figure 4 for an example.","prefix":"s to some location on the ring. ","suffix":" Whena node is added or removed,"}]}]}
>```
>%%
>*%%HIGHLIGHT%%The node that isclosest to the key in the clockwise direction is responsible for that key. See Figure 4 for an example.*
>%%LINK%%[[#^joke9emmkf|show annotation]]
>%%COMMENT%%
>对 key 进行 hash，无需去修改分区的数量，只需对它进行 hash 即可，并将它放在 0 和 1 之间，假设落在一个位置，按照顺时针的顺序遍历这个环，知道找到第一个拥有该数据的节点，这样就知道数据在哪个节点上了。
>%%TAGS%%
>
^joke9emmkf


>%%
>```annotation-json
>{"created":"2022-05-26T09:18:18.725Z","updated":"2022-05-26T09:18:18.725Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":7680,"end":7779},{"type":"TextQuoteSelector","exact":"When a node is added or removed, keys are only moved between nodes adjacent to the new/removed node.","prefix":"y. See Figure 4 for an example. ","suffix":" Areplication factor of n means "}]}]}
>```
>%%
>*%%HIGHLIGHT%%Whena node is added or removed, keys are only moved between nodes adjacent to the new/removed node.*
>%%LINK%%[[#^xglaalz6szs|show annotation]]
>%%COMMENT%%
>当添加新节点的时候，当分布式 DBMS 无法处理那些流量时，就想让它去处理，So想添加新机器，以此来对它进行扩展，使用这种一致性 hash 的话，将服务器添加到环中的某个节点处，唯一需要转移的东西是，B 与 D 区域里的 hash 结果所对应的数据条目，需要从 C 转移到 D 上，集群中的其他分区中的数据仍然还在它们原来的位置。
>%%TAGS%%
>
^xglaalz6szs


![1pqZ4I](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/1pqZ4I.png)


replication factor = 3
对于往数据库中插入的每个 tuple 来说，我们想让这个 tuple 复制到 3 个不同的节点或者是分区上，So，如果其中一个分区挂掉了，还有两个分区可用

如果对A分区进行写入操作，如何知道这个写操作已经被传播到 F 和 B 呢？
必须等待，直到收到它们已经拿到这个写操作的通知，但这可能会比较糟糕，因为其中一个分区可能会挂掉，当在等待这些通知的时候，就会停在那里，或者，假设不想进行等待，但遇上了这个问题，可能要对 A 分区进行写操作，接着立刻去试着读取 B 分区的数据，这可能并不是一致的，但我们希望那个它是一直到的。


==Logical Partitioning VS Physical Partitioning==
>%%
>```annotation-json
>{"created":"2022-05-26T10:41:43.352Z","updated":"2022-05-26T10:41:43.352Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":8138,"end":8294},{"type":"TextQuoteSelector","exact":"Logical Partitioning: A node is responsible for a set of keys, but it doesn’t actually store those keys. Thisis commonly used in a shared disk architecture.","prefix":"les in table two into the other.","suffix":"Physical Partitioning: A node is"}]}]}
>```
>%%
>*%%HIGHLIGHT%%**Logical Partitioning**: A node is responsible for a set of keys, but it doesn’t actually store those keys. Thisis commonly used in a shared disk architecture.*
>%%LINK%%[[#^c110a5d8blo|show annotation]]
>%%COMMENT%%
>将数据库中的某一部分按某种算法分配给这些不同的计算节点
>%%TAGS%%
>
^c110a5d8blo


>%%
>```annotation-json
>{"created":"2022-05-26T10:41:50.110Z","updated":"2022-05-26T10:41:50.110Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":8294,"end":8449},{"type":"TextQuoteSelector","exact":"Physical Partitioning: A node is responsible for a set of keys, and it physically stores those keys. This iscommonly used in a shared nothing architecture.","prefix":"d in a shared disk architecture.","suffix":"5 Distributed Concurrency Contro"}]}]}
>```
>%%
>*%%HIGHLIGHT%%**Physical Partitioning**: A node is responsible for a set of keys, and it physically stores those keys. This iscommonly used in a shared nothing architecture.*
>%%LINK%%[[#^rxqrqsuun1|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^rxqrqsuun1

![[distributed#^20p7rw8clua]]

# Distributed Concurrency Control

![fUdAKj](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/fUdAKj.png)

![tSE3Gj](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/tSE3Gj.png)

## Centralized corrdinator
>%%
>```annotation-json
>{"created":"2022-05-26T10:50:35.129Z","updated":"2022-05-26T10:50:35.129Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":8610,"end":8730},{"type":"TextQuoteSelector","exact":"The centralized coordinator acts as a global “traffic cop” that coordinates all the behavior. See Figure 5 fora diagram.","prefix":"dination.Centralized coordinator","suffix":"MiddlewareCentralized coordinato"}]}]}
>```
>%%
>*%%HIGHLIGHT%%The centralized coordinator acts as a global “traffic cop” that coordinates all the behavior. See Figure 5 fora diagram.*
>%%LINK%%[[#^lauw4s4b68f|show annotation]]
>%%COMMENT%%
>所有人都会跑到某个中心，这个地方可以看到系统中发生的一切，它会判断是否允许进行提交
>%%TAGS%%
>
^lauw4s4b68f

>[!faq] 在上面情况下，提交事务是不安全的
>假设违反了完整性约束，事务就中止了

## Middleware


>%%
>```annotation-json
>{"created":"2022-05-26T11:01:47.368Z","updated":"2022-05-26T11:01:47.368Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":8740,"end":8861},{"type":"TextQuoteSelector","exact":"Centralized coordinators can be used as middleware, which accepts query requests and routes queries tocorrect partitions.","prefix":"Figure 5 fora diagram.Middleware","suffix":"Decentralized coordinatorIn a de"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Centralized coordinators can be used as middleware, which accepts query requests and routes queries tocorrect partitions.*
>%%LINK%%[[#^gmu1qf2bwht|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^gmu1qf2bwht

## Decentralized corrdinator
节点会组织它们自己作出判断，比如这个事务是否能做这些修改，我们是否能提交该事务，我们可以通知涉及该事务的其他人，他们已经成功提交了这个事务。


>%%
>```annotation-json
>{"created":"2022-05-26T11:03:50.804Z","updated":"2022-05-26T11:03:50.804Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 22 Introduction to Distributed Databases","link":[{"href":"urn:x-pdf:046b6ce58bbdad0744a9670f98687d89"},{"href":"vault:/database/15445/note/22-distributed_note.pdf"}],"documentFingerprint":"046b6ce58bbdad0744a9670f98687d89"},"uri":"vault:/database/15445/note/22-distributed_note.pdf","target":[{"source":"vault:/database/15445/note/22-distributed_note.pdf","selector":[{"type":"TextPositionSelector","start":8942,"end":9156},{"type":"TextQuoteSelector","exact":"The client directly sends queries to one of thepartitions. This home partition will send results back to the client. The home partition is in charge ofcommunicating with other partitions and committing accordingly.","prefix":"ach, nodes organize themselves. ","suffix":"Centralized approaches give way "}]}]}
>```
>%%
>*%%HIGHLIGHT%%The client directly sends queries to one of the partitions. This home partition will send results back to the client. The home partition is in charge of communicating with other partitions and committing accordingly.*
>%%LINK%%[[#^x45gf6m82go|show annotation]]
>%%COMMENT%%
>application Server 会和分区中的某些服务器通信，主节点用来对给定事务进行响应，能和 application Server 进行通信的都是主节点，如果在 homogeneous architecture 下，可以将所有查询请求直接发送到主节点或单个分区，但当提交时，就必须去主节点，并说我做了这些更改，想进行提交事务操作，然后主节点负责与其他分区进行通信并确定是否允许提交。主节点只会知道一些有关你接触过的分区的潜在信息，但不知道你对它们做了什么
>%%TAGS%%
>
^x45gf6m82go

如何才能确定它是否可以安全的提交？
