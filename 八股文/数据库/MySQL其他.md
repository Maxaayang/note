## MySQL内部构造可以分为哪两部分

- 服务器层包括连接器、查询缓存、分析器、优化器、执行器等，涵盖了MySQL的大部分核心功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等等
- 存储引擎负责数据的存储和提取。其架构模式是插件式的，支持InnoDB、MyISAM、Memory等多个存储引擎，现在最常用的存储引擎是InnoDB、它从MySQL5.5.5 版本开始成为了默认的存储引擎

## 数据库引擎InnoDB与MyISAM的区别

**InnoDB**
- 是MySQL默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其他存储引擎
- 实现四个标准的隔离级别，默认级别是可重复读。在可重复读隔离级别下，通过多版本控制(MVCC) + 间隙锁(Next-Key Locking)防止幻影读
- 主索引是聚簇索引，在索引中保存了数据，从而避免了直接读取磁盘，因此对查询性能有很大的提升
- 内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够快速插入操作的插入缓冲区等
- 支持真正的在线热备份，其他存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取

**MyISAM**
- 设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它
- 提供了大量的特性，包括压缩表、空间数据索引等
- 不支持事务
- 不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入则对表加排他锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入

MyISAM适合于插入不频繁，查询非常频繁，如果执行大量的SELECT，MyISAM是更好的选择，没有事务。
InnoDB适合于可靠性要求比较高，或者要求事务；表更新和查询都相当的频繁，大量的INSERT或UPDATE

## 数据库为什么进行分库和分表？都放在一个库或者一张表里不可以吗？

分库与分表的目的是减小数据库的单库单表负担，提高查询性能，缩短查询时间

**分表**，可以减少数据库的单表负担，将压力分散到不同的表上，同时因为不用表上的数据量少了，起到提高查询性能，缩短查询时间的作用，此外，可以很大的缓解表锁的问题。分表策略可以归纳为垂直拆分和水平拆分
- 垂直分表：当某个表中的字段比较多，可以新建立一张扩展表，将不经常使用或者长度较大的字段拆分出去放到扩展表中
	- 可以一定程度上避免跨页的问题
	- 可以解决表与表之间的IO竞争
- 水平分表：将表中不同的数据行按照一定规律分布到不同的数据库表中，这样来降低单表数据量，优化查询性能，常见的方式是通过主键或者时间等字段进行Hash和取模后拆分
	- 可以降低单表的数据量，一定程度上可以缓解查询性能瓶颈，但本质上这些表还保存在同一个库中，所以库级别还是会有IO瓶颈
	- 可以解决单表中数据量增长出现的压力

**库内分表**，仅仅是解决了单表数据过大的问题，但并没有把单表的数据分散到不同的物理机上，因此并不能减轻MySQL服务器的压力，仍然在同一个物理机上的资源竞争和瓶颈，包括CPU、内存、磁盘IO、网络带宽等

**分库**，按照业务模型来划分出不同的数据库，而不是像早期一样将所有的数据表都放到同一个数据库中

分库与分表带来的问题：
- 数据迁移与扩容的问题
	- 解决方案：通过程序先读出数据，然后按照指定的分表策略再将数据写入到各个分表中。分页与排序问题
- 分页与排序问题：需要在不同的分表中将数据进行排序并返回，并将不同分表返回的结果集进行汇总和再次排序，最后再返回给用户

## 视图的作用是什么？可以更改吗

视图是虚拟的表，与包含数据的表不一样，视图只包含使用时动态检索数据的查询；不包含任何列或数据。使用视图可以简化复杂的SQL操作，隐藏具体的细节，保护数据；视图创建后，可以使用与表相同的方式利用他们

视图不能被索引，也不能有关联的触发器或默认值，如果视图本身内有 order by，则对视图再次 order by 将被覆盖

对于某些视图比如未使用联结子查询分组聚集函数等，是可以对其更新的，对视图的更新将对基表进行更新；但是视图主要用于简化检索，保护数据，并不用于更新，而且大部分视图都不可以更新。

## 数据库的三大范式
### 第一范式

在任何一个关系数据库中，第一范式是对关系模式的基本要求，不满足第一范式的数据库就不是关系数据库。所谓第一范式是指数据库表的每一列都是不可分割的基本数据项，同一列中不能有多个值，即实体中的某个属性不能有多个值或者不能有重复的属性。

如果出现重复的属性，就可能需要定义一个新的实体，新的实体由重复的属性构成，新实体与原实体之间为一对多关系。在第一范式中表的每一行值包含一个实例的信息。

间而言之，第一范式就是无重复的列。

### 第二范式

第二范式是在第一范式的基础上建立起来的，即满足第二范式必须先满足第一范式。第二范式要求数据库表中的每个实例或行必须可以被唯一地区分

为实现区分通常需要为表加上一个列，以存储每个实例的唯一标识。这个唯一属性列被称为主关键字或主键、主码。第二范式要求实体的属性完全依赖于主关键字

所谓完全依赖是指不能存在仅依赖主关键字一部分的属性，如果存在，那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体，新实体与原实体之间是一对多的关系，为实现区分通常需要为表加上一个列，以存储各个实例的唯一标识。

简而言之，第二范式就是非主属性非部分依赖于主关键字

### 第三范式

满足第三范式必须先满足第二范式。简而言之，第三范式要求一个数据库表中不包含已在其他表中已包含的非主关键字信息。

简而言之，第三范式就是属性不依赖于其他非主属性。

## MySQL中CHAR与VACHAR的区别

- char的长度是不可变的，用空格填充到指定长度的大小，而varchar的长度是可变的
- char的存取速度要bivarchar快很多
- char的存储方式是：对英文字符占用一个字节，对一个汉字占用两个字节。varchar的存储方式是：对每个英文字符占用两个字节，汉字占用两个字节

## SQL语法中的内连接、自连接、外连接、交叉连接的区别

内连接：只有两个元素相匹配的才能在结果集中显示
外连接：
- 左外连接：左边为驱动表，驱动表的数据全部显示，匹配表的不匹配的不会显示。
- 右外连接：右边为驱动表，驱动表的数据全部显示，匹配表的不匹配的不会显示。
- 全外连接：连接的表中不匹配的数据会全部显示出来
- 交叉连接：笛卡尔效应，显示的结果是链接表数的乘积

## 数据库结构的优化手段

- 范式优化：比如消除冗余
- 反范式优化：比如适当加冗余
- 限定数据的范围：务必禁止不带任何限制数据范围条件的查询语句
- 读/写分离：经典数据库拆分方案，主库负责写，从库负责读
- 拆分表：分区将数据在物理上分隔开，不同分区的数据可以制定保存在处于不同磁盘上的数据文件里。这样当对这个表进行查询时，只需要在表分区中进行扫描，而不必进行全表扫描，明显缩短了查询时间，另外处于不同磁盘的分区也将对这个表的数据传输分散在不同的磁盘IO，一个精心设置的分区可以将数据传输对磁盘IO竞争均匀的分散开。对数据量大的实时表可以采取此方法。可按月自动建表分区

## InnoDB 存储格式
### Compact

![OWhxov](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/OWhxov.png)

各变长字段数据占用的字节数按照列的顺序逆序存放，变长字段长度列表中只存储值为非NULL的列内容占用的长度，值为NULL的列的长度是不存储的

NULL值列表：如果表中没有允许存储NULL的列，则NULL值列表也不存在了，否则将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列
- 二进制位的值为1时，代表该列的值为 NULL
- 二进制位的值为0时，代表该列的值不为NULL
NULL值列表必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补0

记录头信息：由固定的5个字节组成
![sqQSYM](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/sqQSYM.png)
![SvFzEZ](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/SvFzEZ.png)

记录的真实数据(隐藏列)
![qiZD7C](https://cdn.jsdelivr.net/gh/Maxaayang/pic@main/uPic/qiZD7C.png)
InnoDB存储引擎会为每条记录都添加transaction_id 和 roll_pointer 这两个列，但是 row_id 是可选的(在没有自定义主键以及Unique键的情况下才会添加该列)