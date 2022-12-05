在shell中赋值的时候不能使用空格

- $1 会访问第一个参数
- $0 是脚本的名字
- $2-9 是脚本的第二到九个参数
- $# 给定的参数个数
- $ $ 这个命令的进程ID
- $? 获取上条命令的错误代码
- $_ 获取上条命令的最后一个参数
- $@ 展开所有参数


```shell
foo=bar

echo "Value is $foo"
# Value is bar
echo 'Value is $foo'
# Valse is $foo, 单引号不会进行替换
```

```shell
mcd.sh

mcd () {
	mkdir -p "$1"
	cd "$1"
}
```
source mcd.sh
mcd test

```shell
# 可以使用逻辑运算符
false || echo "Oops fail"

# 在同一行内可以使用分号来连接命令
false ; echo "This will always print"
# This will always print
```

```shell
# 将命令的输出存到变量里
foo=$(pwd)
echo "We are in $(pwd)"

# 过程替换, 内部的命令会被执行, 其输出将被存储到临时文件内, 然后将文件标识符交给最左边的命令
cat <(ls) <(ls ..)
```


```shell
example.sh

#!/bin/bash

echo "Starting program at $(date)" # Date will be substituded

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
	# 输出到/dev/null的东西会被丢弃掉
	# 2> 是重定向标准错误流的
	grep foobar "$file" > /dev/null 2> /dev/null
	# When pattern is not found, grep has exit status 1
	# We redirect STDOUT and STDERR to a null register, since we do not care about them
	if [[ {"$?" -ne 0 ]]; then
		echo "File $file does not have any foobar, adding one"
		echo "# foobar" >> "$file"
	fi
done
```
./example.sh mcd.sh script.py example.sh

```shell
*.sh

# ? 标记只会展开一个字符
project?

# 转换图片的格式
convert image.png image.jpg
convert image.{png,jpg}

touch foo{,1,2,10}
# touch foo foo1 foo2 foo10

# 笛卡尔积
touch project{1,2}/src/test/test{1,2,3}.py
touch {foo,bar}/{a..j}

# 给出warning和语法错误，以及没有正确引用的地方，或者空格打错的地方
shellcheck mcd.sh
```

- `tldr` 帮助文档
- `fg` 查找工具
- `find . -name main -type d` 查找目录
- `find . -path '**/test/*.py' -type f`
- `find . -mtime -1 ` 最近一天被修改过的东西
- `find . -name "*.tmp" -exec rm {} \;` 对所有找到的文件执行rm命令
- `fd` 另一种查找
- `locate` 查找文件系统中具有指定子串的路径
- `grep -R foobar` 遍历查找整个目录，可以使用rg代替
- `rg "import requests" -t py -C 5 ~/scratch` 结果附近的5行
- `rg -u --files-without-match "^#\!" -t sh` -u是不忽略隐藏文件，只搜索.sh结尾的文件

- Ctrl + r 倒着往前搜索
- `fzf` 模糊搜索，默认绑定到 Ctrl + r 上
- `broot` 查看目录下的文件
