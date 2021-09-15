---
title: Linux实验二笔记
date: 2021-05-31 19:43:47
tags:
categories:
- Linux
---

# 利⽤SSH客户端登录 root 账号，查看 /tmp ⽬录下是否存在⼦⽬录 myshare，如果没有则建⽴该⽬录

```bash
ls /tmp/myshare
//如果没有
mkdir -p /tmp/myshare
```

# 在 myshare ⽬录下创建⼀个名为“学号”的⽂件夹和⼀个名为 exam2.txt 的⽂件

```bash
mkdir /tmp/myshare/学号
touch /tmp/myshare/exam2.txt
```

# 创建⼀个名字为 test 的新⽤户，并指定uid为1024

```bash
useradd -u 1024 test
```

# 把 /etc/passwd 和 /etc/shadow 含有⽤户 test 信息的 ⾏ 追加到 exam2.txt ⽂件中

```bash
cat /etc/passwd | grep test >> /tmp/myshare/exam2.txt
cat /etc/shadow | grep test >> /tmp/myshare/exam2.txt
```

# 把 /etc/passwd 前13⾏的内容 追加到 myshare ⽬录下 名为 exam2.txt 的⽂件中

```bash
head -13 /etc/passwd >> /tmp/myshare/exam2.txt
```

# 把 myshare ⽬录下的所有⽂件和⼦⽬录的内容以⻓格式的⽅式追加到 exam2.txt 中

```bash
ls -Rl /tmp/myshare >> /tmp/myshare/exam2.txt
```

# 把 myshare ⽬录及其⽬录下的所有⽂件和⼦⽬录的拥有者设置为⽤户 test ，组改为mail

```bash
chown -R test:mail /tmp/myshare
```

# 把 myshare ⽬录下的所有⽂件和⼦⽬录的内容以⻓格式的⽅式追加到 exam2.txt 中

```bash
ls -Rl /tmp/myshare >> /tmp/myshare/exam2.txt
```

# 利⽤su命令切换到⽤户 test 账号

```bash
su test
```

# 进⼊/tmp/myshare/“学号”⽬录，采⽤vi编辑器编写以下程序,程序名称为hello.sh：

```shell
\#!/bin/bash
echo "app start"
echo -e
func (){
 echo "hello Linux!"
}
func
echo -e
echo "app end"
```

命令：

```bash
cd /tmp/myshare/学号
vim hello.sh
//输入内容
:wq
```

# 保存 hello.sh 后，给予 hello.sh 拥有者可读、可写和可执⾏的权限，同组可读可执⾏，其他⼈可执⾏权限

```bash
chmod 751 /tmp/myshare/学号/hello.sh
```

# 以⻓格式的形式查看 hello.sh 的⽂件权限信息，并把输出内容追加到 exam2.txt

```bash
ls -l /tmp/myshare/学号/hello.sh >> /tmp/myshare/exam2.txt
```

# 输⼊ ./hello.sh 执⾏脚本，查看输出结果。并把输出结果追加 exam2.txt

```bash
移动到/tmp/myshare/学号/hello.sh
./hello.sh
./hello.sh >> ../exam2.txt
```

# 进⼊⽤户 test 的⽤户主⽬录，在这个⽬录下创建 hello.sh 的软链接myhello.sh，同时拷⻉ hello.sh 到该⽬录下并改名为 hello.sh.bak

```bash
cd ~
ln -s /tmp/myshare/学号/hello.sh myhello.sh
cp /tmp/myshare/学号/hello.sh hello.sh.bak
```

# 以⻓格式形式查看⽤户 test 主⽬录下的所有⽂件并把结果追加到 exam2.txt中

```bash
ls -al ~ >> /tmp/myshare/exam2.txt
```

# 执⾏⽤户 test 主⽬录下的myhello.sh⽂件，查看链接是否正常

```bash
./myhello.sh
```

# 退出⽤户 test 帐号，回到root帐号

```bash
exit
```

# 以⻓格式形式查看⽤户 test 主⽬录下的所有⽂件(含隐藏⽂件)并把结果追加到 exam2.txt中

某个用户的主目录名字通常与用户名相同，并且放在home目录内。

```bash
ls -al /home/test >> /tmp/myshare/exam2.txt
```

# 从 /usr 开始查找后缀名为.conf的所有⽂件(普通⽂件)，把输出结果追加到 exam2.txt中

```bash
find /usr -name "*.conf" -type f >> /tmp/myshare/exam2.txt
```

# 从上⼀步找到的conf⽂件中找出⽂件容量最⼤的⽂件，并把这个⽂件以⻓格式形式追加到exam2.txt 中 （倒引号）

-S表示按大小进行排序，第一条为最大的。

```bash
ls -Sl `find /usr -name "*.conf" -type f` | head -1 >> /tmp/myshare/exam2.txt
```

# 统计出系统中有多少个⽤户帐号，把数量追加到 exam2.txt 中

```bash
cat /etc/passwd |wc -l >> /tmp/myshare/exam2.txt
```

# 把 exam2.txt ⽂件转换为windows格式。

```bash
unix2dos /tmp/myshare/exam2.txt
```

# 把 exam2.txt 传送到window下

```bash
sz /tmp/myshare/exam2.txt
```

# 删除⽤户test 的所有内容（包括主⽬录）

```bash
userdel -r test
```

# 删除/tmp/myshare⽬录

```bash
rm -rf /tmp/myshare
```

# 把 exam2.txt 重命名为 学号.txt 然后提交

```bash
mv exam2.txt 学号.txt
```

