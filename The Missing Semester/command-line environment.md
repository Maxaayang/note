```shell
ctrl + c SIGINT
ctrl + \ SIGQUIT
ctrl + z suspended, 停止并处于后台

SIGKILL 无法被软件捕获, 无论如何它都会中止进程的执行, 并且有时可能是有害的, 所以尽量不要使用它, 因为可能会留下孤立子进程

# & 表示希望程序开始在后台运行
# fg 让程序重新回到终端
# 多个程序可以直接使用 && 来进行运行
# jobs 查看运行的进程
# bg 查看后台运行的进程
nohub sleep 2000 &
bg %1

# 继续暂停任务1
kill -STOP %1

# 为什么要使用nohub命令？
# 如果向第一个作业发送挂断命令，它会以与其他信号类似的方式将其挂断并终止作业, 如果使用nohup的话，在收到 -HUP 的任何地方都会忽略它，并且只是忽略它，以便可以继续运行，如果使用 -KILL 的话，无论如何都会杀死这个作业
kill -HUP %1
```

```shell
# 等号周围不能有空格, 因为alias接受一个参数, 如果有空格就会是多个参数, 从而不会起作用, 因为这不是期望的格式
alias ll="ls -lah"

# 软链接
ln -s 源文件 目标文件

```

```shell
scp notes.md addr@10.249.47.15:foobar.md

# 在断开连接之后rsync会从断开的地方继续复制
rsync -avP . addr@10.249.47.15:cmd

```

