# 进程管理

## 进程
![[Pasted image 20220826112751.png]]
ELF文件
- ==可重定位文件==
![[Pasted image 20220826113017.png]]

- .text：放编译好的二进制可执行代码
- .data：已经初始化好的全局变量
- .rodata：只读数据，例如字符串常量、const 的变量
- .bss：未初始化全局变量，运行时会置 0
- .symtab：符号表，记录的则是函数和变量
- .strtab：字符串表、字符串常量和变量名

局部变量是放在栈里边，是在程序运行过程中随时分配空间，随时释放

可重定位：.o文件里的位置是不确定的，但是必须可以重新定位。

要想使一个函数作为库文件使用，不能以.o的形式存在，必须要形成库文件

**静态链接库**：将一系列对象文件(.o)归档为一个文件
缺点：相同的代码段，如果被多个程序使用，在内存中就有多份，而且一旦静态链接库被更新了，如果二进制文件不重新编译，也不随着更新

```shell
ar cr libstaticprocess.a process.o
gcc -o staticcreateprocess createprocess.o -L. -lstaticprocess
```

-L表示在当前目录下找.a文件， -lstaticprocess会自动补全文件名

- ==可执行文件==

![[Pasted image 20220826185041.png]]
section是多个.o文件合并过的
p_vaddr：这个段被加载到内存中的虚拟地址
e_entry：位于ELF头里边，也是一个虚拟地址，是这个程序运行的入口

- 共享对象文件

**动态链接库**：不仅仅是一组对象文件的简单归档，而是多个对象文件的重新组合，可被多个线程共享
程序文件不包括动态链接库中的代码，仅仅包括对动态链接库的引用，并且不保存动态链接库的全路径，仅仅保存动态链接库的名称

```shell
gcc -shared -fPIC -o libdynamicprocess.so process.o
gcc -o dynamiccreateprocess createprocess.o -L. -ldynamicprocess
```

多了一个.interp的Segment，里面是ld-linux.so，即动态链接器，还多了.plt，过程链接表，.git.plt，全局偏移量表

运行时如何去找到函数？
在二进制程序里，调用PLT[x]里面的代理代码，这个代码会在运行时找真正的函数
GOT会为函数创建一项GOT[y]，是运行时函数真正的地址
对于函数，GOT一开始会创建一项GOT[y]，但是这里边没有真正的地址，它又会回调PLT，PLT再调用PLT[0]，PLT[0]转而调用GOT[2]，里边是ld-linux.so的入口函数，这个函数会找到加载到内存中的libdynamicprocess.so里面的函数的地址，然后把这个地址放在GOT[y]里面，下次，PLT[x]的代理函数就可以直接调用了。

ps -ef 查看当前系统启动的进程
PID 1 的进程是init进程systemd
PID 2 的进程是内核线程kthreadd
用户态的不带中括号，内核态的带中括号
tty为问号的一般都是后台的服务

## 线程

使用进程并行的问题
- 创建进程占用的资源太多
- 进程之间的通信需要数据在不同的内存空间传来传去，无法共享

```C++
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_OF_TASKS 5

void *downloadfile(void *filename)
{
   printf("I am downloading the file %s!\n", (char *)filename);
   sleep(10);
   long downloadtime = rand()%100;
   printf("I finish downloading the file within %d minutes!\n", downloadtime);
   pthread_exit((void *)downloadtime);
}

int main(int argc, char *argv[])
{
   char files[NUM_OF_TASKS][20]={"file1.avi","file2.rmvb","file3.mp4","file4.wmv","file5.flv"};
   pthread_t threads[NUM_OF_TASKS];
   int rc;
   int t;
   int downloadtime;

   pthread_attr_t thread_attr;
   pthread_attr_init(&thread_attr);
   pthread_attr_setdetachstate(&thread_attr,PTHREAD_CREATE_JOINABLE);

   for(t=0;t<NUM_OF_TASKS;t++){
     printf("creating thread %d, please help me to download %s\n", t, files[t]);
     rc = pthread_create(&threads[t], &thread_attr, downloadfile, (void *)files[t]);
     if (rc){
       printf("ERROR; return code from pthread_create() is %d\n", rc);
       exit(-1);
     }
   }

   pthread_attr_destroy(&thread_attr);

   for(t=0;t<NUM_OF_TASKS;t++){
     pthread_join(threads[t],(void**)&downloadtime);
     printf("Thread %d downloads the file %s in %d minutes.\n",t,files[t],downloadtime);
   }

   pthread_exit(NULL);
}
```

![[Pasted image 20220826192042.png]]

PTHREAD_CREATE_JOINABLE：表示将来主线程等待这个线程的结束，并获取退出时的状态。

### 线程的数据

1. 线程上的本地数据
	局部变量，保存在栈里边，每个线程都有自己的栈空间
	栈的大小通过 ulimit -a 查看，通过 ulimit -s 修改
	为了避免线程之间的栈空间踩踏，线程栈之间会有一个小块区域，用来隔离保护各自的栈空间，一旦另一个线程踏入到这个隔离区，就会引发段错误

```C++
// 修改线程栈的大小
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
```

2. 在整个进程里共享的全局数据
	全局变量

3. 线程私有数据

```C++
// 创建线程私有数据
int pthread_key_create(pthread_key_t *key, void (*destructor)(void*))

// 设置key对应的value
int pthread_setspecific(pthread_key_t key, const void *value)

// 获取key对应的value
void *pthread_getspecific(pthread_key_t key)
```

key一旦被创建，所有的线程都可以访问它，但各线程可根据自己的需要往key中填入不同的值

### 数据的保护
1. Mutex(互斥)
	- pthread_mutex_t声明锁
	- 使用pthread_mutex_init初始化mutex
	- pthread_mutex_lock()抢锁，抢到了就继续执行，否则就阻塞
	- pthread_mutex_trylock抢锁，抢到了就继续执行，否则返回一个错误码
	- pthread_mutex_unlock释放锁
	- pthread_mutex_destroy销毁锁

2. 条件变量：释放锁的时候去通知大家
	- pthread_cond_t声明条件变量


## 进程数据结构

由双向循环链表将所有的task_struct(进程描述符)串起来，进程描述符包含的数据能够完整地描述一个正在执行的程序：它打开的文件、进程的地址空间、挂起的信号、进程的状态...

task_struct中的任务ID
```C++
pid_t pid;
pid_t tgid;
struct task_struct *group_leader; 
```

信号处理
```C++
/* Signal handlers: */
struct signal_struct    *signal;
struct sighand_struct    *sighand;
sigset_t      blocked;
sigset_t      real_blocked;
sigset_t      saved_sigmask;
struct sigpending    pending;
unsigned long      sas_ss_sp;
size_t        sas_ss_size;
unsigned int      sas_ss_flags;
```

任务状态
```C++
 volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
 int exit_state;
 unsigned int flags;

/* Used in tsk->state: */
#define TASK_RUNNING                    0
#define TASK_INTERRUPTIBLE              1
#define TASK_UNINTERRUPTIBLE            2
#define __TASK_STOPPED                  4
#define __TASK_TRACED                   8
/* Used in tsk->exit_state: */
#define EXIT_DEAD                       16
#define EXIT_ZOMBIE                     32
#define EXIT_TRACE                      (EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->state again: */
#define TASK_DEAD                       64
#define TASK_WAKEKILL                   128
#define TASK_WAKING                     256
#define TASK_PARKED                     512
#define TASK_NOLOAD                     1024
#define TASK_NEW                        2048
#define TASK_STATE_MAX                  4096
```

- TASK_RUNNING：进程时刻准备运行
- TASK_INTERRUPTIBLE：可中断的睡眠状态
- TASK_UNINTERRUPTBLE：不可中断的睡眠状态
- TASK_KILLABLE：可以终止的睡眠状态
- TASK_TRACED：进程被debugger等进程监视，进程执行被调试程序所停止

运行统计信息
```C++
u64        utime;//用户态消耗的CPU时间
u64        stime;//内核态消耗的CPU时间
unsigned long      nvcsw;//自愿(voluntary)上下文切换计数
unsigned long      nivcsw;//非自愿(involuntary)上下文切换计数
u64        start_time;//进程启动时间，不包含睡眠时间
u64        real_start_time;//进程启动时间，包含睡眠时间
```

进程亲缘关系
```C++
struct task_struct __rcu *real_parent; /* real parent process */
struct task_struct __rcu *parent; /* recipient of SIGCHLD, wait4() reports */
struct list_head children;      /* list of my children */
struct list_head sibling;       /* linkage in my parent's children list */
```

- parent指向父进程，当它终止时，必须向它的父进程发送信号
- children表示链表的头部，链表中的所有元素都是它的子进程
- sibling用于把当前进程插入到兄弟链表中，是一个有环的双向链表，用于组织兄弟进程，每个兄弟进程都会加入到这个双向环链表中

大多数情况下，parent和real_parent是一样的
特殊情况：bash创建了一个进程，进程的parent和real_parent都是bash，如果在bash上使用gdb来debug一个进程，这个时候gdb就是parent，bash是这个进程的real_parent

项目组权限
```C++
/* Objective and real subjective task credentials (COW): */
const struct cred __rcu         *real_cred;
/* Effective (overridable) subjective task credentials (COW): */
const struct cred __rcu         *cred;
```

- real_cred：谁能操作我
- cred：我能操作谁

RCU：Linux的一种同步机制，(读、拷贝、更新)，级随意读，但是在更新数据的时候，需要先复制一份副本，在副本上完成修改，再一次性地替换旧数据，是Linux实现的一种读多写少的共享数据的同步机制

```C++
struct cred {
......
        kuid_t          uid;            /* real UID of the task */
        kgid_t          gid;            /* real GID of the task */
        kuid_t          suid;           /* saved UID of the task */
        kgid_t          sgid;           /* saved GID of the task */
        kuid_t          euid;           /* effective UID of the task */
        kgid_t          egid;           /* effective GID of the task */
        kuid_t          fsuid;          /* UID for VFS ops */
        kgid_t          fsgid;          /* GID for VFS ops */
......
        kernel_cap_t    cap_inheritable; /* caps our children can inherit */
        kernel_cap_t    cap_permitted;  /* caps we're permitted */
        kernel_cap_t    cap_effective;  /* caps we can actually use */
        kernel_cap_t    cap_bset;       /* capability bounding set */
        kernel_cap_t    cap_ambient;    /* Ambient capability set */
......
} __randomize_layout;
```

用户态函数栈

![[Pasted image 20220826203008.png]]
A调用B，A的栈里包含A函数的局部变量，然后是调用B的时候要传递给它的参数，然后返回A的地址，这就是A的栈帧
B的栈帧先保存A的栈帧的栈底位置，因为在B函数里面获取A传进来的参数，就是通过这个指针获取的，接下来是B的局部变量等

![[Pasted image 20220826203618.png]]
内核态函数栈
thread_info是对task_struct的一个补充，因为task_struct结构庞大但是通用，不同的体系结构需要补充不同的东西，所以往往与体系结构有关的，都放在thread_info中

## 调度

- 实时进程：需要尽快返回结果
- 普通进程

对于实时进程，优先级范围是0~99；对于普通进程，优先级范围是100~139，数值越小，优先级越高

==实时调度策略==
- SCHED_FIFO：高优先级的线程可以抢占第优先级的进程，而相同优先级的进程遵循先来先得
- SCHED_RR：轮流调度算法，相同优先级的任务当用完时间片会被放到队列尾部，而高优先级的任务而是可以抢占低优先级的任务
- SCHED_DEADLINE：按照任务的deadline进行调度，DLc调度器总是选择其deadline距离当前时间点最近的那个任务，并调度它执行

==普通调度器==
- SCHED_NORMAL：普通的进程
- SCHED_BATCH：后台进程，可以默默地执行，不要影响需要交互的进程，可以降低它的优先级
- SCHED_IDLE：特别空闲的时候才跑的进程

```C++
// 调度策略的执行逻辑
const struct sched_class *sched_class;
```

- stop_sched_class：优先级最高的任务会使用这种策略
- dl_sched_class：对应deadline调度策略
- rt_sched_class：对应RR算法或FIFO算法的调度策略，具体调度策略是由进程的task_struct->policy指定
- fair_sched_class：普通进程的调度策略
- idle_sched_class：空闲进程的调度策略

### 完全公平调度算法(CFS)

理想情况下，每个进程可以运行相同的时间，但是这是不现实的，因为无法在一个处理器上真的同时运行多个进程，而且如果每个进程运行无限小的时间周期也是不高效的，因为调度时进程抢占会带来一定的代价，将一个进程换出，另一个进程换入本身有消耗，同时还会影响缓存的效率

==实际做法==：允许每个进程运行一段时间、循环轮转、选择运行最少的进程作为下一个运行进程，而不采用分配给每个进程时间片的做法，越高的nice，进程获得更低的处理器使用权重，更低的nice值的进程获得更高的处理器使用权重。
绝对的nice值并不影响调度决策，只有相对的nice值才影响处理器时间的分配比例

CFS需要一个数据结构来对运行时间来进行排序，这个数据结构不仅需要在查询的时候能够快速找到最小的，更新的时候也需要能够快速调整排序。所以使用的是==红黑树==，可以平衡查询和更新速度

==红黑树放在哪？==
每个CPU都有自己的struct rq结构，其用于描述在此CPU上运行时所有的进程，其包括一个实时进程队列rt_rq和一个CFS运行队列cfs_rq，在调度时，调度器首先会去实时队列中找是否有实时进程需要运行，如果没有才会去CFS运行队列找是否有进程需要运行。

如何平衡CPU之间的负载？
- Linux内核会在每一个时钟周期末尾触发rebalabce
- cpu发现自己队列为空也会去摘取其他cpu的任务

最小粒度：每个进程获得的时间片的底线，默认1ms
为了防止可运行的任务趋近于无限时，各自所获得的处理器使用比和时间片都趋近于0
CFS并不是完美的公平调度：因为处理器的时间片再小也无法突破最小粒度

- 任何进程所获得的处理器时间是由它自己和其他所有可运行的进程nice值的相对差值决定的
- nice值对时间片的作用不再是算数加权，而是几何加权
- 任何nice值对应的绝对时间不再是一个绝对值，而是处理器的使用比

### 主动调度

上下文切换
- 切换进程空间，即虚拟内存
- 切换寄存器和CPU上下文

寄存器和栈的切换
tss，在x86体系结构中，提供了一种以硬件的方式进行进程切换的模式，对于每个进程，x86希望在内存中维护一个TSS(任务状态段)结构，里面有所有的寄存器
TR(任务寄存器)，指向某个进程的TSS。更改TR的值，将会触发硬件保存CPU所有寄存器的值到当前进程的TSS中，然后从新进程的TSS中读出所有寄存器值，加载到CPU对应的寄存器中。


## 进程的创建

copy_process()
- dup_task_struct()
	- 调用alloc_task_struct_node分配一个task_struct结构
	- 调用alloc_thread_stack_node创建内核栈，里边调用__vmalloc_node_range分配一个连续的THREAD_SIZE的内存空间，赋值给task_struct的void \*stack成员变量
	- 调用arch_dup_task_struct(struct task_struct \*dst, struct task_struct \*src)，将task_struct进行复制
	- 调用setup_thread_stack设置thread_info


## 线程的创建

线程不是一个完全由内核实现的机制，它是由内核态和用户态合作完成的


# 内存管理

最低位先是Text Segment、Data Segment和BSS Segment
Text Segment：存放可执行文件
Data Segment：存放静态常量
BSS Segment：存放未初始化的静态常量

然后是堆段
堆是往高地址增长的，用来动态分配内存，malloc就是在这里分配的

Memory Mapping Segment：用来把文件映射进内存，如果二进制的执行文件依赖于某个动态链接库，就是在这个区域内将so文件映射到了内存中

栈地址段：主线程的函数调用的函数所在栈所在地

在内核空间里边所有进程看到的都是同一个内核空间，看到的都是同一个进程列表，但内核栈是各用各的，所以如果要访问一些公共数据结构的话，需要进行锁保护

Linux中所有的段的起始地址都是一样的，都是0，所以并没有用到全部的分段功能，但分段可以做权限审核

![[Pasted image 20220827145204.png]]

虚拟地址分为页号和页内偏移量

32位环境下，每个页表项需要4字节来存储
页表项必须提前建好，并且是连续的，如果不连续的话，就无法通过虚拟地址里面的页号来找到对应的页表项了，并且因为需要快速定位页表项，即需要可以随机访问，因此需要连续，对应的数据结构就需要使用数组

可以将页表再次进行分页，即页目录，这样虽然总的需要的内存变大了，但是不用一次进行全部的分配，所以实际使用的内存就不会那么大了

对于64位的系统，变成了四级的目录：全局目录项PGD、上层目录项PUD、中间目录项PMD、页表项PTE

## 进程空间管理

虚拟内存空间可以分为**用户态地址空间**和**内核态地址空间**















