---
layout: post
title:  "Linux基础信息"
date:   2017-10-13 21:03:00 +0800
categories: linux
tags: [linux, 资料]
---

* TOC
{:toc}

# Linux Filesystem Hierarchy

```shell

/
├── bin # User Binaries 用户可执行程序
├── sbin # System Binaries 系统二进制文件
├── boot # Boot Loader Files 引导加载程序文件
├── dev # Device Files 设备文件
├── etc # Configuration Files 配置文件
├── home # Home Directories 家目录
├── lib # System Libraies 系统库
├── media # Removable Devices 可移动媒体设备
├── mnt # Mount Directory 挂载目录
├── opt # Optional add-on Apps 可选的附加应用程序
├── proc # Process Infomation 进程信息
├── srv # Service Data 服务数据
├── tmp # Temporary Files 临时文件
├── usr # User Programs 用户程序
├── var # Variable Files 变量文件
├── vmlinuz -> boot/vmlinuz-4.4.0-89-generic
├── vmlinuz.old -> boot/vmlinuz-4.4.0-85-generic
├── root
├── run
├── sys
├── lib64
├── lost+found
├── initrd.img -> boot/initrd.img-4.4.0-89-generic
├── initrd.img.old -> boot/initrd.img-4.4.0-85-generic
```
# 命令

## 命令帮助
* man ps

## 用户
* `finger username`  显示用户username的信息
* `who`              显示当前登陆用户
* `who am I`
* `su`                成为root用户
* `sudo command`      以root用户身份执行
* `passwd`            更改密码

## SHELL
* `history`                  显示在当前shell下命令历史
* `alias`                      显示所有的命令别称
* `alias new_command='command'`    将命令command别称为new_command
* `env`                       显示所有的环境变量
* `export var=value`    设置环境变量var为value

## 显示硬盘、分区、CPU、内存信息
* `df -lh`                           显示所有硬盘的使用状况
* `du -sh *`                       显示当前目录下各个目录和文件的大小
* `mount`                          显示所有的硬盘分区挂载
* `mount partition path`       挂在partition到路径path
* `umount partition`            卸载partition
* `sudo fdisk -l`                  显示所有的分区
* `sudo fdisk device`            为device(比如/dev/sdc)创建分区表。 进入后选择n, p, w
* `sudo mkfs -t ext3 partition`  格式化分区patition(比如/dev/sdc1)修改 /etc/fstab，以自动挂载分区。增加行：/dev/sdc1  path(mount point) ext3 defaults 0 0
* `arch`                            显示架构
* `cat /proc/cpuinfo`          显示CPU信息
* `cat /proc/meminfo`         显示内存信息
* `free`                            显示内存使用状况
## 网络
* `ifconfig`      显示网络接口以及相应的IP地址。ifconfig可用于设置网络接口
* `ifup eth0`    运行eth0接口
* `ifdown eth0`  关闭eth0接口
* `iwconfig`      显示无线网络接口
* `route`        显示路由表。route还可以用于修改路由表
* `netstat`      显示当前的网络连接状态
* `ping IP`      发送ping包到地址IP
* `traceroute IP` 探测前往地址IP的路由路径
* `dhclient`      向DHCP主机发送DHCP请求，以获得IP地址以及其他设置信息。
* `host domain`  DNS查询，寻找域名domain对应的IP
* `host IP`      反向DNS查询
* `wget url`      使用wget下载url指向的资源
* `wget -m url`  镜像下载

## 进程
* `top`              显示进程信息，并实时更新
* `ps`                显示当前shell下的进程
* `ps -lu username`  显示用户username的进程
* `ps -ajx`          以比较完整的格式显示所有的进程
* `kill PID`         杀死PID进程 (PID为Process ID)

## 文件 

* `touch filename`    如果文件不存在，创建一个空白文件；如果文件存在，更新文件读取和修改时间。
* `rm filename`      删除文件
* `cp file1 file2`    复制file1为file2
* `ls -l path`        显示文件和文件相关信息
* `mkdir dir`        创建dir文件夹
* `mkdir -p path`    递归创建路径path上的所有文件夹
* `rmdir dir`        删除dir文件夹，dir必须为空文件夹。
* `rm -r dir`        删除dir文件夹，以及其包含的所有文件
* `file filename`    文件filename的类型描述
* `chown username:groupname filename`    更改文件的拥有者为owner，拥有组为group
* `chmod 755 filename`更改文件的权限为755: owner r+w+x, group: r+x, others: r+x 
* `od -c filename`    以ASCII字符显示文件
* `cat filename`      显示文件
* `cat file1 file2`  连接显示file1和file2
* `head -1 filename`  显示文件第一行
* `tail -5 filename`  显示文件倒数第五行
* `diff file1 file2`  显示file1和file2的差别
* `sort filename`    对文件中的行排序，并显示
* `sort -f filename`  排序时，不考虑大小写
* `sort -u filename`  排序，并去掉重复的行
* `uniq filename`    显示文件filename中不重复的行 (内容相同，但不相邻的行，不算做重复)
* `wc filename`      统计文件中的字符、词和行数
* `wc -l filename`    统计文件中的行数