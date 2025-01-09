---
title: Linux 文件目录与权限
date: 2018-12-29 21:26:21
tags:
categories: Linux
---

## 1.目录树

如果我们将整个目录树以图标的方法来显示，并且将较为重要的文件数据列出来的话，那么目录树架构有点像这样：

![LinuxFileDirectories](/images/LinuxFileDirectories.gif)

## 2.处理目录的常用命令

```
ls: 列出目录
cd：切换目录
pwd：显示目前的目录
mkdir：创建一个新的目录
rmdir：删除一个空的目录
cp: 复制文件或目录
rm: 移除文件或目录
```

例如:

```
#-r：递归持续复制，用於目录的复制行为；(常用)
cp -r test test1
#-r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！
rm -rf test
```

## 3.文件权限
1. 使用ll或者ls –l命令来显示一个文件的属性以及文件所属的用户和组
    
```
[root@www /]# ls -l
    total 64
    dr-xr-xr-x   2 root root 4096 Dec 14  2012 bin
    dr-xr-xr-x   4 root root 4096 Apr 19  2012 boot
```

2. 在Linux中第一个字符代表这个文件是目录、文件或链接文件等等

```
当为[ d ]则是目录
    当为[ - ]则是文件；
    若是[ l ]则表示为链接文档(link file)；
    若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
    若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。
```

3. 接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合

其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。
每个文件的属性由左边第一部分的10个字符来确定（如下图）。

![LinuxFilePermission](/images/LinuxFilePermission.png)

4. 文件的权限字符为：『-rwxrwxrwx』

这九个权限是三个三个一组，分别是owner/group/others三种身份各有自己的read/write/execute权限。数字来代表各个权限，各权限的分数对照表如下：

```
r:4
w:2
x:1
```

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为： [-rwxrwx---] 分数则是：

    
```
owner = rwx = 4+2+1 = 7
group = rwx = 4+2+1 = 7
others= --- = 0+0+0 = 0
```

5. 所以我们设定权限的变更时：

```
chmod [-R] xyz 文件或目录
#例如
chmod 777 .bashrc
```