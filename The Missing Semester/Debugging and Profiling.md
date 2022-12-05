```shell
cat /var/log/system.log | lnav

# 将日志记录到系统日志中
logger "HEllo Logs"
```

```shell
# ipdb
p locals()   # 返回所有值的字典

# 磁盘使用情况
du
```

core dumped

`valgrind`可以跟踪程序的一些堆栈信息
valgrind -v 可执行程序名字

- 当程序程序出现了段错误时, Linux内核会根据配置情况将一个`core dump`文件写入到硬盘中.
-   Linux用`ulimit`设置连接数的最大值, `ulimit`只能做临时修改,重启后失效:
    -   `ulimit -c` 设置core文件的最大值, 单位为区块;
    -   `ulimit -a` 显示目前资源限制的设定.
    -   利用`ulimit -c unlimited`将core文件设置为无限大.

-   不能产生core文件的原因:
    -   没有足够内存空间;
    -   禁用了core文件的创建;
    -   设置一个进程当前目录没有写文件的的权限;

- 利用命令`sudo sysctl -w kernel.core_pattern=/tmp/core-%e.%p.%h.%t`设置内核产生core文件的形式和位置, 放于`/tmp`目录并且显示时间戳.
    -   当程序出现段错误的时候, linux内核会自动地在`/tmp`目录保存一个core文件.

-   利用`cat /proc/PID/limit`也可以显示一个进程中的core文件的大小限制.

-   `kernel.core_pattern`表示`coredumps`文件放于什么地方,它是一个内核参数,可以通过`sysctl`进行查看和进行控制:
    -   `sysctl -a`表示查看内核的所有参数, 或使用`sysctl kernel.core_pattern`显示`kernel.core_pattern`的参数.

通过GDB工具对生成的core文件进行回溯追踪

-   通过命令`gdb -c ./test my_core_file`打开一个名为`my_core_file`的文件.


sudo sysctl -w kernel.core_pattern=/home/ubuntu/mini/core-%e.%p.%h.%t
sudo sysctl -w kernel.core_pattern=/root/source/miniob/core-%e.%p.%h.%t
sudo sysctl -w kernel.core_pattern=/home/ljj/project/miniob/core-%e.%p.%h.%t

sudo sysctl -w kernel.core_pattern=/home/ljj/project/addr/build/polarkv/test/core-%e.%p.%h.%t

https://zhuanlan.zhihu.com/p/39736407
https://www.cnblogs.com/longjiang-uestc/p/10635135.html

![[Pasted image 20221017220639.png]]