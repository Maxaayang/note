# project one



# project two

## 在 `hash_table_bucket_page` 中定义 `occupied_` 的时候为什么要减一？
```C++
// TODO(max): 这里为什么要减一?

char occupied_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1];

// 0 if tombstone/brand new (never occupied), 1 otherwise.

char readable_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1];
```

为了节省空间，如果需要 39 个位，直接 39/8 + 1 = 5，但是如果是 40 的话，计算完之后就变为 6 了，多出了一个不需要的 char

## 在 `extendible_hash_table` 中的 `KeyToDirectoryIndex` 进行读取的时候为什么不需要加写锁？

因为在调用这个函数之前，其他函数已经加了 Write/Read Lock，所以不需要再次进行加锁，否则可能会导致无法申请到锁而进行阻塞。




# project three



## insert

- `insert` 与 `update` 都没有使用函数 `Next` 自带的 `tuple` 与 `rid`，因为这两个操作之后是不需要将数据进行返回的。
- 





## update

- 为什么在 NEXT 函数中不使用函数自己的 `tuple` 和 `rid` ？

  因为在测试程序 `executor_test.cpp` 中，会在 `update` 之后检查是否将结果进行了返回，如果使用函数自己的 `tuple` 就会将结果进行返回。

  ```C++
  if (result_set != nullptr && tuple.IsAllocated()) {
    result_set->push_back(tuple);
  }
  ```



# project four




# Database Storage




# Lecture #05: Buffer Pools

- 空间控制策略(Spatial  Control)：通过决定将 pages 写到磁盘的哪个位置，使得常常一起使用的 pages 能离得更近，从而提高 I/O 效率。
- 时间控制策略(Temporal Control)：通过决定何时将 pages 读入内存，写回磁盘，使得读写的次数最小，从而提高 I/O 效率。

## Locks  vs.  Latches

locks 是数据库系统中的抽象锁
-   可以保护数据库中的内容，例如tuple、表等。
-   事务在运行时会持有这个lock。
-   交易期间持有需要能够回滚更改。——后面并发控制等章节会讨论
-   数据库系统会为使用数据库的人提供这个API或者函数，为程序开发人员使用。

latches 是操作系统中的锁
-   用来保护DBMS的关键部分与其他线程之间的互斥。
-   保护DBMS在运行期间进行独享。
-   不需要能够回滚更改。

## Buffer Pool

DBMS 启动时会从 OS 申请一片内存区域，即 Buffer Pool，并将这块区域划分成大小相同的 pages，为了与 disk pages 区别，通常称为 frames，当 DBMS 请求一个 disk page 时，它首先需要被复制到 Buffer Pool 的一个 frame 中，如下图所示：

![PoRAlX](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/PoRAlX.png)

同时 DBMS 会维护一个 page table，负责记录每个 page 在内存中的位置，以及是否被写过（Dirty Flag），是否被引用或引用计数（Pin/Reference Counter）等元信息，如下图所示：

![l8Wlcc](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/l8Wlcc.png)

当 page table 中的某 page 被引用时，会记录引用数（pin/reference），表示该 page 正在被使用，空间不够时不应该被移除；当被请求的 page 不在 page table 中时，DBMS 会先申请一个 latch（lock 的别名），表示该 entry 被占用，然后从 disk 中读取相关 page 到 buffer pool，释放 latch。以上两种过程如下图所示：

![ZdQL71](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/ZdQL71.png)

将磁盘中的 page 拷贝到 buffer 池的 frame 中的顺序是任意的，不一定与磁盘中 page 的顺序是一致的。

page table 所做的事情：
- 记录每个 page 在内存中的位置
- Dirty Flag，记录 page 是否被修改过，以及追踪是被谁进行了修改==如何追踪被谁进行了修改？==
- 引用计数。

page table 本质是 hash table，如果想要找一个 page 的话，通过 page table 和 page id 就可以知道该 page 在哪个 frame 中。

**Memory  Allocation  Policies**
- Global  policies：同时考虑所有查询来分配内存
-  local policies：为单个查询分配内存时不考虑其他查询的情况


## Buffer Pool Optimizations
### Multiple Buffer Pools

为了减少并发控制的开销以及利用数据的 locality，DBMS 可以在不同维度上维护多个 Buffer Pools：
- 多个 Buffer Pools 实例，相同的 page hash 到相同的实例上。
- 每个 DataBase 分配一个 Buffer Pool。
- 每种 Page 类型一个 Buffer Pool。

使用 Multiple Buffer Pools 可以在每个 buffer pool 上使用局部策略，这样就可以为放入的数据进行量身定制，选择最好的局部策略，并且这样可以减少 latch 的竞争。

![qwIyNu](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/qwIyNu.png)

### Pre-fetching

DBMS 通过 query plan 来预取 pages
- Sequential Scans
- Index Scans

### Scan  Sharing

主要用在多个查询存在数据共用的情况。当两个查询 A，B 先后发生，B 发现自己有一部分数据与 A 共用，于是就会先共用 A 的cursor，等 A 扫描完之后，再扫描自己还需要的其他数据。

### Buffer Pool  Bypass

当遇到扫描量非常大的查询时，如果将所需的pages从磁盘中一个一个地换入 Buffer Pool，将会造成 buffer 的污染，因为这些 pages 通常只使用一次，而它们的进入将导致一些可能在未来更需要的 pages 被移除。

解决方案：分配一小块内存该那个线程，当它从磁盘中读取 page 时，不管这个 page 是否已经在 buffer 池中了，都要放在这一小块内存中，当查询完成时就会释放掉这块内存，这样就不会对 buffer 池造成污染。并且这样也可以避免对 page table 进行查询所带来的开销。

## OS Page  Cache

大部分 disk operations 都是通过系统调用完成，通常系统会维护自身的数据缓存，这会导致一份数据分别在操作系统和 DMBS 中被缓存两次。大多数 DBMS 都会使用 (O_DIRECT) 来告诉 OS 不要缓存这些数据，除了 Postgres。

> [!faq] 为什么要这样做？
> 1. 因为数据库缓冲池中有一份副本，而操作系统也缓存了一份磁盘 page 副本，在我们更新数据库缓冲池中的 page 后，操作系统缓存的那份 page 就是旧的，显然这是一份多余的数据，作为数据库系统，我们希望自主管理我们的 page，不想让操作系统掺合。
> 2. . 不同操作系统的 page 缓存策略也是不同的，同一种数据库有 Linux 也有 Windows 版本，为了保证跨操作系统之间的一致性，这也需要数据库自己本身管理一切。

## Buffer  Replacement Policies
### Least Recently  Used  (LRU)

维护每个 page 上一次被访问的时间戳，每次移除时间戳最早的 page。

### CLOCK

Clock 是 LRU 的近似策略，它不需要每个 page 上次被访问的时间戳，而是为每个 page 保存一个 reference bit
-   每当 page 被访问时，reference bit 设置为 1   
-   每当需要移除 page 时，从上次访问的位置开始，按顺序轮询每个 page 的 reference bit，若该 bit 为 1，则重置为 0；若该 bit 为 0，则移除该 page

为什么是近似？
因为点那个需要移除的 page 时，不会去精确地移除最近最少使用的那个 page。

==缺点==：sequential flooding(顺序洪水)，执行某种特殊的操作时会连续将 page 换入，这会导致我们需要的 page 被从缓冲池中移除掉，与 Buffer Pool Bypass 的问题一致。

![1I1qrZ](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/1I1qrZ.png)

解决措施：
- LRU-K：保存每个 page 的最后 K 次访问时间戳，利用这些时间戳来估计它们下次被访问的时间，通常 K 取 1 就能获得很好的效果。
-   localization  per  query：针对每个查询做出移除 pages 的限制，使得这种影响被控制在较小的范围内，类似 API 的 rate limit。
-  priority hints ：有时候 DBMS 知道每个 page 在查询执行过程中的上下文信息，因此它可以根据这些信息判断一个 page 是否重要。

### Dirty  Pages

移除一个 dirty page 的成本要高于移除一般 page，因为前者需要写 disk，后者可以直接 drop，因此 DBMS 在移除 page 的时候也需要考虑到这部分的影响。除了直接在 Replacement Policies 中考虑，有的 DBMS 使用 **Background Writing** 的方式来处理。它们定期扫描 page table，发现 dirty page 就写入 disk，在 Replacement 发生时就无需考虑脏数据带来的问题。

# Lecture #06:  Hash Tables





# Lecture #07: Tree Indexes






# Lecture #08: Index Concurrency Control

- Logical  Correctness：是否可以看到应该要看到的数据？
- Physical  Correctness：数据的内部是否完好？如何保证线程读取数据时该数据结构的可靠性？

## Latch Modes

![[projects and notes#Locks vs Latches|Locks  vs.  Latches]]

Read Mode
- 多个线程可以同时读取相同的数据
- 针对相同的数据，当别的线程已经获得处于 read mode 的 latch，新的线程也可以继续获取 read mode 的 latch

Write Mode
- 同一时间只有单个线程可以访问
- 针对相同的数据，如果获取前已经有别的线程获得任何 mode 的 latch，新的线程就无法获取 write mode 的 latch

## Hash Table Latching

不会发生死锁，因为所有的线程都是从同一个方向来获取数据，当要改变 hash table 的大小的时候，只需要加一个全局的锁即可。

### page latches

缺点：会减少并发，因为一个 page 在同一时间只能被一个线程锁获取。
优点：如果一个线程在一个 page 中同时获取多个 slot 的话，会变得更快。

### slot latches

优点：可以增加并发，因为不同的线程可以同时在同一个 page 中获取不同的 slot。
缺点：增加了访问的存储和计算开销，因为线程必须为它们访问的每个插槽获取一个锁存器，并且每个插槽必须存储闩锁的数据。DBMS可以使用单一模式锁存器(即Spin锁存器)来减少元数据和以一些并行性为代价的计算开销。

## B+Tree  Latching

我们希望在最大程度上允许多个线程同时读取、更新同一个 B+ Tree index，主要需要考虑两种情况：
-   多个线程同时修改同一个 node
-   一个线程正在遍历 B+ Tree 的同时，另一个线程正在 splits/merges nodes

### Latch  crabbing/coupling

- 获取 parent 的 latch
- 获取 child 的 latch
- 如果**安全**，则可以释放 parent 的 latch

安全指的是当发生更新操作时，该节点不会发生 split 或 merge 的操作，即：
- 在插入元素时，节点未满
- 在删除元素时，节点超过半满

在实际的应用中：
- 更新操作每次都需要在路径上获取 write latch 容易称为系统并发的瓶颈
- 通常 Node 的大小为一个 page，可以存储很多 keys，因此更新操作的出现频率不算高

缺点：事务在进行 Insert/Delete 时总是会在 root 获取写锁，这样会限制并发度。
改进：假设 leaf node 是安全的，那么就在 root 申请读锁，当到达 leaf node 时是安全的，那么直接申请 write latch 然后更新数据，否则就中止事务并重新开始，此时在 root 处直接申请 write latch。

### Leaf Node  Scans

在 Leaf Node  Scans 中可能会发生死锁，但是 B+ Tree 的锁天然不支持死锁检测。
如何解决？
当一个线程获取叶子结点的锁时失败了，那么就直接中止，并重新开始。

### Delay Parent Updates

每当 leaf node 溢出时，都需要至少更新 3 个节点：
- 即将被拆分的 leaf node
- 新的 leaf node
- parent node

修改的成本较高，因此 B-link Tree 提出了一种优化策略：每当 leaf node 溢出时，只是标记一下而暂时不更新 parent node，当下一次有别的线程获取 parent node 的 write latch 时，一并修改。

==这种方式只可以用在 insert 操作中==


# Lecture #9: Sorting & Aggregation Algorithms
[Sorting & Aggregation Algorithms](https://zhenghe.gitbook.io/open-courses/cmu-15-445-645-database-systems/sorting-and-aggregations#external-merge-sort)
## Sorting

**Why Do We Need To Sort?**
需要排序算法的原因：**本质在于 tuples 在 table 中没有顺序**，无论是用户还是 DBMS 本身，在处理某些任务时希望 tuples 能够按一定的顺序排列，如：
-   若 tuples 已经排好序，去重操作将变得很容易（DISTINCT)
-   批量将排好序的 tuples 插入到 B+ Tree index 中，速度更快
-   Aggregations (GROUP BY)

### Algorithms

若数据能够放入内存中，我们可以使用标准排序算法搞定，如快排；若数据无法放入内存中，就得考虑数据在 disk 与 memory 中移动的成本，以及排序算法的适配。

#### External Merge Sort

外部排序通常有两个步骤：
-   Sorting Phase：将数据分成多个 chunks，每个 chunk 可以完全读入到 memory 中，在 memory 中排好序后再写回到 disk 中
-   Merge Phase：将多个子文件合并成一个大文件



# Lecture #16: Two-Phase Locking

> [!faq] 为什么加锁和放锁要泾渭分明地分为两个阶段呢？  
2PL并发控制目的是为了达到 serializable，如果并发控制不事先将所有需要的锁申请好，而是释放锁后，还允许再次申请锁，可能出现事务内两次操作同一对象之间，其它事务修改这一对象，进而无法达到conflict serializable，出现不一致的现象。

[DataBase · 理论基础 · 热点优化 (SIGMOD'21 Paper 解读)](http://mysql.taobao.org/monthly/2022/02/03/)

在实际的系统中，除非预先知道事务的整个处理流程，否则很难知道在什么时间点之后，该事务不会再读写数据（也就是不会再请求新的锁），也就是说第一阶段结束的时间点无法确定。 所以，通常的做法是，事务完成整个处理流程之后，再进入到释放阶段开始释放锁。


# Lecture #17: Timestamp Ordering Concurrency Control

TimeStamp ordering(T/O) 是一个乐观的并发控制策略。在事务访问数据时不需要显式地加锁。每个事务都会被分配一个唯一的时间戳 TS(T<sub>i</sub>)。如果 TS(T<sub>i</sub>) < TS(T<sub>j</sub>)，数据库就必须保证 schedule 的执行是与先执行 T<sub>i</sub> 再执行 T<sub>j</sub> 是等价的。

## 时钟方案

- system lock：会在像夏令这样的极端情况下出现问题。
- logical counter：在分布式系统中维持计数器会导致溢出。
- hybrid：

## Basic TimeStamp Ordering(BASIC T/O)

每条数据都会有两条标记：
- R-TS(X)：最后一次读X发生的时间戳
- W-TS(X)：最后一次写X发生的时间戳
在每个事务中，DBMS都会对操作进行检查，看是否读取或写入了未来的数据，一旦发现就会中止并进行重启。

### Read Operations

如果 TS(T<sub>i</sub>) < W-TS(X)，那么就会中止并重启事务，在重启事务之后会为事务分配一个新的时间戳（==否则会遇到上次的问题，必须保证时间戳是单调增加的==）。
如果事务没有进行重启的话，就会更新 R-TS(X)， 并且复制一份数据 X 来保证可重复读。

### Write Operations

如果 TS(T<sub>i</sub>) < R-TS(X) or TS(T<sub>i</sub>) < W-TS(X)，事务就必须被重启，反之意味着它在修改过去的数据，符合规范。在进行修改之后，它还需要去把数据 X 拷贝一份来保证在这个事务结束之前可重复读。

### Optimization: Thomas Write Rule

如果遇到修改未来的数据，DBMS会忽略写操作，并且让事务继续执行而不是中止并进行重启。（==因为事务 T<sub>i</sub> 所写的东西永远凑不会被读取到==）

如果不使用 TWR 优化，Basic T/O 能够生成 conflict serializable 的 schedule。如果使用了 TWR，则 Basic T/O 生成的 schedule 虽然与顺序执行的效果相同，但不满足 conflict serializable。

### Basic T/O Advantage

- 不会产生死锁，因为没有事务会等待。
- 如果单个事务涉及的数据不多、不同事务涉及的数据基本不相同 (OLTP)，可以节省 2PL 中控制锁的额外成本，提高事务并发度。

### Basic T/O Disadvantage

- starvation：长事务容易因为短事务的冲突而饿死，因为长事务的timestamp偏小，大概率会在执行一段时间后读到更新的数据，导致abort。
- 复制数据，维护、更新时间戳存在额外成本。
- 可能产生不可恢复的 schedule (具体见下节)
- 在高并发的场景下会出现时间戳分配的瓶颈。

## Optimistic Concurrency Control (OCC)

OCC 也是一种乐观的并发控制策略，


# Lecture #18: Multi-Version Concurrency Control

**MVCC的优点**：
- 写不会阻塞读
- 读不会阻塞写

只读事务无需加锁就可以读取数据库某一时刻的快照，如果保留数据的所有历史版本，DBMS 甚至能够支持读取任意历史版本的数据，即 time-travel。

MVCC并不是可串行化的。

## Concurrency Control Protocol

[浅析数据库并发控制机制](http://catkang.github.io//2018/09/19/concurrency-control.html)
- Timestamp Ordering (T/O)：为每个事务赋予时间戳，并用以决定执行顺序
- [[projects and notes#Write Operations|Optimistic Concurrency Control (OCC)]]：为每个事务创建 private workspace，并将事务分为 read, write 和 validate 3 个阶段处理
- Two-Phase Locking (2PL)：按照 2PL 的约定获取和释放锁，==并不能解决死锁==。



## Version Storage

如何存储一条数据的多个版本？DBMS 通常会在每条数据上拉一条版本链表 (version chain)，所有相关的索引都会指到这个链表的 head，DBMS 可以利用它找到一个事务应该访问到的版本。不同的版本存储方案在 version chain 上存储的数据不同，主要有 3 种存储方案：

- **Approach 1**：Append-Only Storage：新版本通过追加的方式存储在同一张表中 
- **Approach 2**：Time-Travel Storage：老版本被复制到单独的一张表中 
- **Approach 3**：Delta Storage：老版本数据的被修改的字段值被复制到一张单独的增量表 (delta record space) 中

### Append-Only Storage

如下图所示，同一个逻辑数据的所有物理版本都被存储在同一张表上，每次更新时，就往表上追加一个新的版本记录，并在旧版本的数据上增加一个指针指向新版本：
![[Pasted image 20220620152728.png]]
再次更新的行为类似：
![Xz0Uoo](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/Xz0Uoo.png)

也许你已经注意到，指针的方向也可以从新到旧，二者的权衡如下：

Approach 1：Oldest-to-Newest (O2N)：写的时候追加即可，读的时候需要遍历链表 。Approach 2：Newest-to-Oldest (N2O)：==写的时候需要更新所有索引指针==，读的时候不需要遍历链表。
使用O2N可以直接在链表的末尾添加新的版本，而N2O要将新的版本的指针指向原来的头结点，需要更新全部的索引，使其指向新的版本，因为索引始终是指向头结点的。

### Time-Travel Storage

分别维护 time-travel table 和 main table 
- time-travel table：存储的是旧版本的tuple
- main table：每次更新时把数据复制到 time-travel table 中，然后在 mian table 中写入新的数据，并使用指针指向旧的数据。

![axiIIg](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/axiIIg.png)


### Delta Storage

每次更新，仅将变化的字段信息存储到 delta storage segment 中：
优点：节省了存储的空间。
缺点：在查看历史版本的话需要进行恢复，用时间换空间。

![Au4I3S](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/Au4I3S.png)


## Garbage Collection

随着时间的推移，DBMS 中数据的旧版本可能不再会被用到，如：
-   已经没有活跃的事务需要看到该版本
-   该版本是被一个已经中止的事务创建

这时候 DBMS 需要删除这些可以回收的物理版本，这个过程也被称为 GC。在 GC 的过程中，还有两个附加设计决定：
-   如何查找过期的数据版本
-   如何确定某版本数据是否可以被安全回收

GC 可以从两个角度出发：
- Approach 1：Tuple-level：直接检查每条数据的旧版本数据 
- Approach 2：Transaction-level：每个事务负责跟踪数据的旧版本，DBMS 不需要亲自检查单条数据

### Tuple-level GC

对表进行循序扫描，通过使用时间戳个活跃的事务来搞清这些版本是否过期，不仅需要检查内存中的数据，还需要去检查已经交换到磁盘中的数据。

#### Background Vacuuming

Vacuum 守护线程会周期性地检查每条数据的不同版本，如果它的结束时间小于当前活跃事务的最小时间戳，则将其删除。

![LCuGcf](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/LCuGcf.png)
![6UgDtV](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/6UgDtV.png)

为了加快 GC 的速度，DBMS 可以再维护一个脏页位图 (dirty page bitmap)，利用它，Vacuum 线程可以只检查发生过改动的数据，用空间换时间。Background Vacuuming 被用于任意 Version Storage 的方案。

#### Cooperative Cleaning

当 worker thread 查询数据时，顺便将不再使用物理数据版本删除，==只能用在O2N==。

![3fjuTJ](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/3fjuTJ.png)

如果一个事务发现正在遍历的某个版本对于其他事务都是无视的，就会将这些标记为 deleted，并回收它们所占用的空间（并不是立马去回收），最后会更新索引，让它们指向 version chain 的新头结点。

### Transaction-level GC

让每个事务都保存着它的读写数据集合 (read/write set)，当 DBMS 决定什么时候这个事务创建的各版本数据可以被回收时，就按照集合内部的数据处理即可。

## Index Management
### Primary Key Index

主键索引总是指向 version chain 的头部。
DBMS是否更新主键索引取决于更新一个tuple时系统是否创建一个新的版本。
事务更新一个主键属性时，会先进行删除，然后再进行插入。

### Secondary Indexes
#### Logical Pointers

Key记录的是被索引的列的数据
Value记录的是 Primary Key or Tuple Id。
缺点：查到主键之后需ƒ要去回表，回到主索引去找出整行记录。

![wVhiIl](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/wVhiIl.png)
![VjWNjB](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/VjWNjB.png)
indirection layer 将 tuple 的逻辑 ID 映射到数据库中的物理位置，每当更新 version chain 的时候，只需要更新这个间接层，不需要去更新每个索引。
可以维护一个 TupleId 到 Address 的中间表，更新了版本之后，只需要更新中间表每个行ID指向的物理地址。

#### Physical Pointers

Physical Pointers 存储指向 version chain 头部的指针。

![fW7Oxm](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/fW7Oxm.png)

和主索引一样记录行的物理地址，不用回表，但是每个表的辅助索引很多，每个辅助索引都指向物理地址，当插入一个新的值之后不只是主键索引指向的物理地址需要改，所有的辅助索引指向的物理地址都需要更改。






















