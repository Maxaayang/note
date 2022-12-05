# MySQL技术内幕 InnoDB存储引擎

## MySQL体系结构和存储引擎

数据库：文件的集合

数据库实例：程序，位于用户与操作系统之间的一层数据管理软件（真正用于操作数据库文件的）

### 存储引擎

**mysql区别于其他数据库的特点：插件式的表存储引擎（基于表而不是数据库）**

存储引擎的优点：每个存储引擎都有各自的特点，能够根据具体的应用建立不同的存储引擎表。

### 连接MySQL

本质：进程的通信

常用的进程通信方式：管道、命名管道、命名字、TCP/IP套接字、UNIX域套接字。

* 在通过TCP/IP连接到MySQL实例时，MySQL数据库会先检查一张权限视图，用来判断发起请求的客户端IP是否允许连接到MySQL实例。

* 在Linux与UNIX环境下，可以使用UNIX域套接字（不是一个网络协议），只能在MySQL客户端和数据库实例在同一台服务器上的情况下使用。

  ```MySQL
  show variables like 'socket';		//UNIX域套接字的查找
  
  mysql -udavid -S /tmp/mysql.sock	//UNIX域套接字连接
  ```



# InnoDB存储引擎

目标：面向在线事物处理的应用

特点：行锁设计、支持外键、支持MVCC，并支持类似于Oracle的非锁定读（默认操作不会产生锁），被设计用来最有效地利用以及使用内存和CPU。

## InnoDB 体系架构

后台线程的作用：

1. 刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据
2. 将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下 InnoDB 能恢复到正常运行状态

### 后台线程

InnoDB 存储引擎是**多线程**的模型

#### Master Thread

非常核心的后台线程

主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新，合并插入缓冲、UNDO页的回收等。



#### IO Thread

InnoDB 存储引擎大量使用 AIO 来处理写 IO 请求，从而极大地提高了数据库等性能。

IO Thread 主要负责这些 IO 请求的回调处理

read thread 和 write thread 分别为 4 个

使用 innodb_read_io_threads 和 innodb_write_io_threads 参数进行设置

读线程的 ID 总是小于写线程



#### Purge Thread

作用：事物被提交后，所使用的 undolog 可能不再需要，需要 PurgeThread 来回收已经使用并分配的 undo 页。

purge 可以独立到单独的线程中进行（减轻 Master Thread 的工作，从而提高 CPU 的使用率以及提升存储引擎的性能）

```mysql
//在MySQL数据库的配置文件中添加命令来启用独立的 Purge Thread
[mysqld]
innodb_purge_threads=1
```

InnoDB 支持多个 Purge Thread，可以进一步加快 undo 页的回收，由于 Purge Thread 需要离散地读取 undo 页，也可以进一步利用磁盘的随机读取性能。



#### Page Cleaner Thread

作用：将之前版本中脏页的刷新操作都放入到单独的线程中来完成

目的：减轻原 Master Thread 的工作及对于用户查询线程的阻塞，进一步提高 InnoDB 存储引擎的性能

