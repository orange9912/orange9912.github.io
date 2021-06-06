---
title: Linux实验6笔记
date: 2021-06-03 16:32:03
tags:
categories:
- Linux
---

# 编写一段shell，保存为program.sh

**完成以下输出，可循环执行：**

**5（回车）**

5 4 3 2 1

4 3 2 1

3 2 1 

2 1

1

[分析：用双重循环即可，熟悉语法就可以了]{.pink}

```shell
#!/bin/bash
echo -n "input a number:"
read num
num2=$num
while [ $num2 -gt 0 ]
do
num=$num2
while [ $num -gt 0 ]
do
echo -n "$num "
num=`expr $num - 1`
done
num2=`expr $num2 - 1`
done
```

# 管理员root每天需要完成以下工作：

## 题目

**每天早上8点30分启动服务器的ftp服务，在每天晚上23点30分就关闭ftp服务如果启动成功把ftp的进程信息写⼊/var/ftp/年-⽉-⽇.log ⽂件中，如果启动失败，需要给root发⼀封邮件。邮件内容为： start ftp error。**

**在早上8点30分到晚上23点30分之间，每隔1⼩时ping⼀下百度域名(每次ping 发4次)，保证⽹络畅通，并把ping的结果追加到 /var/ftp/年-⽉-⽇.log ⽂件中。**

**每天晚上11点50分30秒备份ftp⽬录(/var/ftp)⽣成名为 年-⽉-⽇.tar.gz 的压缩包，并把压缩包的权限修改为只有root有读权限，其他都没有权限。把压缩包移动到root主⽬录下。然后清空/var/ftp下的所有内容。假设/var/ftp⽬录已存在**

## 分析

题目所用到的知识点分为两块，一为shell编程，二为定时器crontab。

按照题意，我们可以把要编写的脚本分为四个

**启动ftp服务脚本、关闭ftp服务脚本、ping百度脚本、备份脚本**

### 启动ftp服务脚本

```shell
#!/bin/bash
systemctl start vsftpd
if [ $? != 0 ]
then
mail -s "start ftp error" root
else
processInfo=`ps -ef | grep vsftpd | head -1`
year=`date -I | cut -d - -f1`
month=`date -I | cut -d - -f2`
day=`date -I | cut -d - -f3`
echo "$processInfo" >> /var/ftp/$year-$month-$day.log
fi
```

### 关闭ftp服务脚本

```shell
#!/bin/bash
systemctl stop vsftpd
if [ $? != 0 ]
then
mail -s "close ftp error" root
```

### ping百度脚本

```shell
#!/bin/bash
year=`date -I | cut -d - -f1`
month=`date -I | cut -d - -f2`
day=`date -I | cut -d - -f3`
echo `ping -c4 www.baidu.com` >> /var/ftp/$year-$month-$day.log
```

### 备份脚本

```shell
#!/bin/bash
year=`date -I | cut -d - -f1`
month=`date -I | cut -d - -f2`
day=`date -I | cut -d - -f3`
tar -zcvf $year-$month-$day.tar.gz /var/ftp
chmod 400 $year-$month-$day.tar.gz
rm -r /var/ftp/*
```

### 定时器配置

**cron启动后搜索/var/spool/cron目录，寻找以/etc/passwd文件中的用户名命名的crontab文件，被找到的这种文件将载入内存。比如root用户下的定时器文件则为/var/spool/cron/root**

通常在某用户下想创建定时器，输入crontab -e即可。

```shell
30 8 * * * ~/startFtp.sh
30 23 * * * ~/closeFtp.sh
30 8-23 * * * ~/pingBaidu.sh
50 23 * * * ~/backup.sh
```

