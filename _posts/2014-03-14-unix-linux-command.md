---
layout:         post
title:         Unix/Linux基本命令
description: 本篇介绍Unix/Linux下常用的命令，供参考
keywords: Unix, Linux
category: Unix
tags: [Unix,Linux,Shell]
---

###文件命令

* `ls` - 列出目录
* `ls -al` - 使用格式化列出隐藏文件
* `cd dir` - 更改目录到dir
* `cd` - 更改到home目录
* `cd -` - 返回上一次目录
* `pwd` - 显示当前目录
* `mkdir dir` - 创建目录dir
* `rm file` - 删除file
* `rm -r dir` - 删除目录dir

*加`-f`为强制删除*

<!-- more -->

* `cp file1 file2` - 将file1复制到file2

*加`-r`为复制目录；且目录不存在自动创建*

* `mv file1 file2` - 将file1重命名或者移动到file2；如果file2是一个存在的目录则将file1移到到file2中
* `ln -s file link` - 创建file的符号链接link
* `touch file` - 创建文件file
* `cat > file` - 将标准输入添加到file
* `more file` - 查看file的内容
* `head file` - 查看file的前10行
* `tail file` - 查看file的后10行

*加`-f`表示从后10行开始查看file的内容*


###进程管理

* `ps` - 显示当前的活动进程

*加`–au`为查看系统中，所有使用者的process*
*加`–aux`查看系统中，包含系统内部，所有使用者的process*

* `top` - 显示所有正在运行的进程
* `kill pid` - 杀掉进程id *pid*
* `killall proc` - 杀掉所有名为*proc*的进程 *
* `bg` - 列出已停止或者后台的作业
* `fg` - 将最近的作业带到前台
* `fg n` - 将作业n带到前台

###文件操作

* `chmod octal file` - 更改file的权限

    4 - 读(r)
    2 - 写(w)
    1 - 执行(x)

示例：
    `chmod 777` - 为所有用户添加读、写、执行权限
    `chmod 755` - 为所有者添加rwx权限，为组和其他用户添加rx权限

* `chown usr:pwd file` - 更改文件file所有者为usr
* `chown -R usr:pwd dir` - 更改目录dir所有者为usr
* `diff [-r] name1 name2` - 比较文件或目录之内容, name1 name2可同时为文件名，或目录名称


###SSH

* `ssh usr@host` - 以usr用户身份连接到host
* `ssh -p port user@host` - 在端口prot以user用户身份连接到host


###搜索
* `grep pattern file` - 搜索file中匹配pattern的内容
* `grep -r pattern dir` - 递归搜索dir目录中匹配pattern的内容
* `command | grep pattern` - 搜索command命令输出中匹配pattern的内容

示例：
    `ps aux | grep java` - 搜索匹配java的进程

* `find` - 搜索文件

*加`-name`为按文件名搜索*


###系统信息

* `date` - 显示当前日期和时间
* `cal` - 显示当月的日历
* `uptime` - 显示系统从开机到现在所运行的时间
* `w` - 显示登录的用户
* `whoami` - 显示你的当前用户名
* `finger user` - 显示user的相关信息
* `uname -a` - 显示内核信息
* `cat /proc/cpuinfo` - 查看cpu信息
* `cat /proc/meninfo` - 查看内存信息
* `man command` - 显示command的说明手册
* `df` - 显示磁盘占用情况
* `du` - 显示目录空间占用情况
* `free` - 显示内存及交换区占用情况

###压缩/解压

* `tar cf file.tar files` - 创建包含files的tar文件file.tar
* `tar xf file.tar` - 从file.tar提取文件
* `tar czf file.tar.gz files` - 使用Gzip压缩创建tar文件
* `tar xzf file.tar.gz` - 使用Gzip提取tar文件
* `tar cjf file.tar.bz2` - 使用Bzip2压缩创建tar文件
* `gzip file` - 压缩file并重命名为file.gz
* `gzip -d file.gz` - 将file.gz解压为file

###网络

* `ping host` - ping host并输出结果
* `whois domain` - 获取domain的whois信息
* `dig domain` - 获取domain的DNS信息
* `dig -x host` - 逆向查询host
* `wget file` - 下载file
* `wget -c file` - 断点续传

###文件传输

* `rcp [-r] source hostname:destination` - 拷贝文件或目录至远端工作站

示例：
    `rcp -r dir1 doc:/home/user` - 将目录dir1，拷贝到工作站doc路径/home/user之目录下 

* `rcp [-r] hostname:source destination` - 自远端工作站，拷贝文件或目录

###快捷键

* `Ctrl+C` - 停止当前命令
* `Ctrl+Z` - 停止当前命令，并使用fg恢复
* `Ctrl+D` - 注销当前会话，与`exit`相似
* `Ctrl+W` - 删除当前行中的字
* `Ctrl+U` - 删除整行
* `!!` - 重复上次命令


