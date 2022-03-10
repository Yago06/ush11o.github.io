---
title: Linux常用指令
date: 2020-05-29 19:11:44
tags:
    - Linux
    - 操作指南
categories:
    - 操作指南
---

## 文件操作

### cd

Change Directory之意。
非常基本的一个指令，用于切换当前目录，它的参数是要切换到的目录的路径，可以是绝对路径，也可以是相对路径。

```
cd /root/Docements # 切换到目录/root/Docements
cd ./path          # 切换到当前目录下的path目录中，“.”表示当前目录  
cd ../path         # 切换到上层目录中的path目录中，“..”表示上一层目录
```

### ls

List之意。
这是一个非常有用的查看文件与目录的命令。

```
-l ：列出长数据串，包含文件的属性与权限数据等
-a ：列出全部的文件，连同隐藏文件（开头为.的文件）一起列出来（常用）
-d ：仅列出目录本身，而不是列出目录的文件数据
-h ：将文件容量以较易读的方式（GB，kB等）列出来
-R ：连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来
```

如

```
ls -l filename #以长数据串的形式列出filename文件的数据
ls -l #以长数据串的形式列出当前目录下的数据文件和目录
ls -lR #以长数据串的形式列出当前目录下的所有文件
```

### find

如其名，搜索。是一个非常强大的命令，使用复杂，参数也多。

```
find [PATH] [option] [action]

# 与时间有关的参数：
-mtime n : n为数字，意思为在n天之前的“一天内”被更改过的文件；
-mtime +n : 列出在n天之前（不含n天本身）被更改过的文件名；
-mtime -n : 列出在n天之内（含n天本身）被更改过的文件名；
-newer file : 列出比file还要新的文件名
# 例如：
find /root -mtime 0 # 在当前目录下查找今天之内有改动的文件

# 与用户或用户组名有关的参数：
-user name : 列出文件所有者为name的文件
-group name : 列出文件所属用户组为name的文件
-uid n : 列出文件所有者为用户ID为n的文件
-gid n : 列出文件所属用户组为用户组ID为n的文件
# 例如：
find /home/ljianhui -user ljianhui # 在目录/home/ljianhui中找出所有者为ljianhui的文件

# 与文件权限及名称有关的参数：
-name filename ：找出文件名为filename的文件
-size [+-]SIZE ：找出比SIZE还要大（+）或小（-）的文件
-tpye TYPE ：查找文件的类型为TYPE的文件，TYPE的值主要有：一般文件（f)、设备文件（b、c）、
             目录（d）、连接文件（l）、socket（s）、FIFO管道文件（p）；
-perm mode ：查找文件权限刚好等于mode的文件，mode用数字表示，如0755；
-perm -mode ：查找文件权限必须要全部包括mode权限的文件，mode用数字表示
-perm +mode ：查找文件权限包含任一mode的权限的文件，mode用数字表示
# 例如：
find / -name passwd # 查找文件名为passwd的文件
find . -perm 0755 # 查找当前目录中文件权限的0755的文件
find . -size +12k # 查找当前目录中大于12KB的文件，注意c表示byte
```

### cp

Copy之意。
用于复制文件。常用参数如下。

```
-a ：将文件的特性一起复制
-p ：连同文件的属性一起复制，而非使用默认方式，与-a相似，常用于备份
-i ：若目标文件已经存在时，在覆盖时会先询问操作的进行
-r ：递归持续复制，用于目录的复制行为
-u ：目标文件与源文件有差异时才会复制
```

如

```
cp -a file1 file2 #连同文件的所有特性把文件file1复制成文件file2
cp file1 file2 file3 dir #把文件file1、file2、file3复制到目录dir中
```

### mv

Move之意。
用于移动文件、目录或更名。常用参数如下。

```
-f ：force强制的意思，如果目标文件已经存在，不会询问而直接覆盖
-i ：若目标文件已经存在，就会询问是否覆盖
-u ：若目标文件已经存在，且比目标文件新，才会更新
```

如

```
mv file1 file2 file3 dir # 把文件file1、file2、file3移动到目录dir中
mv file1 file2 # 把文件file1重命名为file2
```

### rm

Remove之意。
用于删除目录或文件。常用参数如下。

```
-f ：就是force的意思，忽略不存在的文件，不会出现警告消息
-i ：互动模式，在删除前会询问用户是否操作
-r ：递归删除，最常用于目录删除，它是一个非常危险的参数
```

如

```
rm -i file # 删除文件file，在删除之前会询问是否进行该操作
rm -fr dir # 强制删除目录dir中的所有文件
```

另：`rm -rf /` 表示强制递归删除根目录，常用于删库跑路。

### cat

concatenate files and print on the standard output.
cat命令的用途是连接文件或标准输入并打印。这个命令常用来显示文件内容，或者将几个文件连接起来显示，或者从标准输入读取内容并显示。

```
cat [option] [file]
```

主要功能

```
cat filename # 一次显示整个文件
cat > file name # 创建一个文件
cat file1 file2 > file # 将几个文件合并成一个文件
```

说实话我没怎么用过，记下来防止看到的时候不懂是什么。

## 其他指令

### Ctrl + C

(kill foreground process)

发送SIGINT信号给前台进程组中的所有进程，强制终止程序的执行。

### Ctrl + Z

(suspend foreground process)

发送 SIGTSTP 信号给前台进程组中的所有进程，常用于挂起一个进程，而并非结束进程，用户可以使用使用fg/bg操作恢复执行前台或后台的进程。fg命令在前台恢复执行被挂起的进程，此时可以使用ctrl-z再次挂起该进程，bg命令在后台恢复执行被挂起的进程，而此时将无法使用ctrl-z再次挂起该进程；

### Ctrl + D

(Terminate input, or exit shell)

输入EOF，相当于终端中输入exit后回车。

### Ctrl + L

清屏


### watch

watch可以帮你监测一个指令的运行结果，省的一遍遍手动运行，他会周期性的执行指令，并全屏显示结果。可以监测一切想要的一切指令的结果变化。

```
watch [option] [order]
```

```
-n 或 --interval # 可以用来置顶间隔的时间（以秒计）
-d 或 --differences # watch会高亮显示变化的区域，而-d=cumulative会把变动过的所有地方都高亮显示出来
-t 或 -no-title # 不显示watch命令在顶部的一大堆信息
-h 或 --help # 显示帮助
```