---
title: Linux实验一笔记
date: 2021-05-31 14:42:55
tags:
categories:
- Linux
---

# 查看虚拟机的ip信息

```javascript
ifconfig
```

# 测试虚拟机与FTP服务器的连通性

```javascript
ping 172.26.14.30
```

# 安装上传下载工具lrzsz

```javascript
yum install -y lrzsz
```

# 查看lrzsz中rz和sz的路径

```bash
which rz sz
```

# 在系统中创建一个txt文件，上传到linux中

```bash
rz
```

# 把这个txt文件移动到/tmp目录下，并重命名为exam1.txt

```bash
mv xxx.txt /tmp/exam1.txt
```

# 把exam1.txt转成unix格式

```bash
yum install -y dos2unix
dos2unix /tmp/exam1.txt
```

# 利用vim编辑器打开exam1.txt，并输入学号、保存退出

```bash
vim exam1.txt
//进入后
i
输入学号
esc
:wq
```

# 利⽤重定向把字符串 1234567890 追加到 exam1.txt 的末尾

```bash
echo "1234567890" >> exam1.txt
```

# 把/etc/passwd的最后5⾏追加到 exam1.txt 中

```bash
tail -5 /etc/passwd >> exam1.txt
```

# 搜索 /usr 下所有以 xml 结尾的⽂件(只搜索普通⽂件)，并把路径中含有 common 的⽂件路径追加到 exam1.txt 中

```bash
find /usr -name "*.xml" -type f | grep common >> exam1.txt
```

# 把当前时间按照 年-⽉-⽇ 时:分:秒 的格式追加到 exam1.txt 中。如：2020-11-23 09:32:43

```bash
date +"%Y-%m-%d %H:%M:%S" >> exam1.txt
```

# 对⽬录 /var/log 进⾏压缩⽣成名为 log.tar.gz ⽂件

```bash
tar -zcvf log.tar.gz /var/log
```

# 通过 ls 命令以⻓格式的形式查看 log.tar.gz 的信息，并把信息追加到 exam1.txt中

```bash
ls -l log.tar.gz >> exam1.txt
```

# 利⽤ awk 命令提取上⼀步中打印的 log.tar.gz 的 ⽂件类型和权限信息 追加到exam1.txt 中

打印某一列用awk，若截取字符串用cut

```bash
ls -l log.tar.gz | awk '{print $1}' >> /tmp/exam1.txtz
```

# 把exam1.txt转成window格式

```bash
unix2dos /tmp/exam1.txt
```

# 把exam1.txt传递到window下

```bash
sz /tmp/exam1.txt
```

