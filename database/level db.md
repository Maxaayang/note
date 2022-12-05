# 基本组件

## 编码

![[Pasted image 20220823145823.png]]

变长编码的目的：减少空间占用
每一个Byte最高位bit用0/1表示该整数是否结束，用剩余7bit表示实际的数值

```C++
char* EncodeVarint32(char* dst, uint32_t v) {
  // Operate on characters as unsigneds
  uint8_t* ptr = reinterpret_cast<uint8_t*>(dst);
  static const int B = 128;
  if (v < (1 << 7)) {
    *(ptr++) = v;
  } else if (v < (1 << 14)) {
    *(ptr++) = v | B;
    *(ptr++) = v >> 7;
  } else if (v < (1 << 21)) {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = v >> 14;
  } else if (v < (1 << 28)) {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = (v >> 14) | B;
    *(ptr++) = v >> 21;
  } else {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = (v >> 14) | B;
    *(ptr++) = (v >> 21) | B;
    *(ptr++) = v >> 28;
  }
  return reinterpret_cast<char*>(ptr);
}
```

## 跳表
![[Pasted image 20220823153126.png]]
==这里应该是先sequence后type==

其中 sequence 为全局自增序列号，LevelDB 遇到一个修改操作，全局序列号自动加一。LevelDB 中的 Key 存储了多个版本的 Value。LevelDB 使用序列号来标记键值对的版本，序列号越大，对应的键值对越新。

type为数据类型，标记是Put还是Delete操作，只取两个值，0表示Delete，1表示Put

如果是删除操作，后面的value_size字段值为0，value字段值为空。将Delete操作等价看成Put操作，同时为了节省空间，internal_key_size和value_size都要采用varint整数编码。

## 内存管理

![[Pasted image 20220823150154.png]]
- 优先判断当前的容量是否足够，如果够的话直接进行分配
- 如果剩余的空间不够
	- 判断需要的空间是否大于 1/4 BlockSize
		- 如果大于的话，分配所需的字节数
		- 否则直接新分配一个块，从新的块中分配一部分内存，之前剩余的内存直接丢弃，主要是为了快速

每次分配一块尽可能大的内存，每次小的内存分配请求都会从这个分配好的内存区域获取，提升了new的性能(不需要brk等系统调用，减少了分配过程中的各种condition race)，同时也减少了内存碎片的概率。

在分配内存的时候如果之前的不够了，就会先判断需要的内存是否大于 1/4 BlockSize，如果小于的话才会分配一个 BlockSize，这样就保证了对于每个block浪费的空间不会超过 1/4 BlockSize

delete只有在arena生命周期结束时，才会统一回收内存，这样也降低了因为漏掉delete/free而造成的memory leak的问题

## 各种Key与Compare

- SequenceNumber：递增的uint64整数，相同的key则按其降序
	- 计算方式：WriteBatch个数 + 1
- kValueTypeForSeek：记录当前操作类型，后续压缩会根据该类型进行对应操作
- 在一个uint64中，type占用一个字节，SeqNum占用7个字节

![[Pasted image 20220823153159.png]]
==这里应该是先sequence后type==

## WriteBatch

![[Pasted image 20220823160423.png]]
上边的是Put，下边的是Delete，因为Delete之后不需要value