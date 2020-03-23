---
title: linux笔记
date: 2019-06-21 09:16:52
tags:
- Linux
categories:
- Linux
- note
---
### 文件操作

##### 列出文件列表

```shell
ls  -a 显示所有   -l 长格式显示
ll
```

##### 目录跳转

```shell
cd / 进入根目录
cd .. 进入上级目录
cd ～ 回到home目录
```
<!--more-->
##### 文件夹（文件）操作

```shell
mkdir 创建文件夹
touch fileName 创建文件
rm -rf fileName/dirName 删除文件或文件夹
chmod 777 fileName 给文件设置权限
```

##### 文件复制 cp

```shell
cp -r a/.  b  复制文件夹
cp aaa/* /bbb 复制目录aaa下所有到/bbb目录下，这时如果/bbb目录下有和aaa同名的文件，需要按Y来确认并且会略过aaa目录下的子目录。
cp -r aaa/* /bbb 这次依然需要按Y来确认操作，但是没有忽略子目录。
cp -r -a aaa/* /bbb 依然需要按Y来确认操作，并且把aaa目录以及子目录和文件属性也传递到了/bbb。 
cp -r -a aaa/* /bbb 成功，没有提示按Y、传递了目录属性、没有略过目录。
```

##### 文件移动mv

```shell
mv 原文件名  新文件名   //修改文件名称
mv  *  ../ 将当前文件夹下所有文件移动到上一层文件夹
```

##### 读取文件 cat

```shell
cat fileName  一次显示整个文件 
cat > filename 从键盘创建一个文件,只能创建新文件,不能编辑已有文件
cat file1 file2 > file


-n 或 --number 由 1 开始对所有输出的行数编号
-b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号
-s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
-v 或 --show-nonprinting
```

##### 查看文件 more，less

```shell
more fileName
+n 从笫n行开始显示
+/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示 

less 主要使用
-b <缓冲区大小> 设置缓冲区的大小
-e  当文件显示结束后，自动离开
-f  强迫打开特殊文件，例如外围设备代号、目录和二进制文件
-g  只标志最后搜索的关键词
-i  忽略搜索时的大小写
-m  显示类似more命令的百分比
-N  显示每行的行号
-o <文件名> 将less 输出的内容在指定文件中保存起来
-Q  不使用警告音
-s  显示连续空行为一行
-S  行过长时间将超出部分舍弃
-x <数字> 将“tab”键显示为规定的数字空格
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）
b  向后翻一页
d  向后翻半页
h  显示帮助界面
Q  退出less 命令
u  向前滚动半页
y  向前滚动一行
空格键 滚动一行
回车键 滚动一页
[pagedown]： 向下翻动一页
[pageup]：   向上翻动一页
```

##### Tail 命令 查看日志

```shell
tail -f 命令可用于监视另一个进程正在写入的文件的增长。 特别是在看日志时非常有用，你实时更新了日志，它就实时显示出来
-n <K> 输出最后K行，K是数字，使用-n + K表示从每个文件的第K行输出
-c <K> 输出最后K行，K是数字，使用-c + K表示从每个文件的第K字节输出
```

##### 文件内容搜索grep

```shell
grep -n searchValue fileName  显示包含searchValue的行，及行号
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行! 
进阶：
grep ‘energywise’ *           #在当前目录搜索带'energywise'行的文件
grep -r ‘energywise’ *        #在当前目录及其子目录下搜索'energywise'行的文件
grep -l -r ‘energywise’ *     #在当前目录及其子目录下搜索'energywise'行的文件，但是不显示匹配的行，只显示匹配的文件
```

##### 查找文件 fin

```shell
find  -name "*.txt" -print 用于查找所有的‘ *.txt’文件在当前目录及子目录中
find  -name "[A-Z]*" -print 用于当前目录及子目录中查找文件名以一个大写字母开头的文件
find /etc -name "host*" -print 在/etc目录中查找文件名以host开头的文件
find  -name "[a-z][a-z][0--9][0--9].txt" -print 在当前目录查找文件名以两个小写字母开头，跟着是两个数字，最后是.txt的文件

使用
1.在某目录下查找名为“elm.cc”的文件
find /home/lijiajia/ -name elm.cc
 
2.查找文件名中包含某字符（如"elm"）的文件
find /home/lijiajia/ -name '*elm*'
find /home/lijiajia/ -name 'elm*'
find /home/lijiajia/ -name '*elm'
 
3.根据文件的特征进行查询
find /home/lijiajia/ -amin -10        #查找在系统中最后10分钟访问的文件
find /home/lijiajia/ -atime -2        #查找在系统中最后48小时访问的文件
find /home/lijiajia/ -empty           #查找在系统中为空的文件或者文件夹
find /home/lijiajia/ -group cat       # 查找在系统中属于groupcat 的文件（试了，命令不对。）
find /home/lijiajia/ -mmin -5         # 查找在系统中最后5 分钟里修改过的文件
find /home/lijiajia/ -mtime -1        #查找在系统中最后24 小时里修改过的文件
find /home/lijiajia/ -nouser          #查找在系统中属于作废用户的文件（不明白是什么意思）
find /home/lijiajia/ -amin 10         #查找在系统中最后10分钟访问的文件
find /home/ftp/pub -user lijiajia     #查找在系统中属于lijiajia这个用户的文件
(PS:以上都是在 /home/lijiajia/文件夹下进行的操作)
 
4.使用混合查找方式查找文件
find /tmp -size +10000000c -and -mtime +2      #查找/tmp目录中大于10000000字节并且在48小时内修改的某个文件
find /tmp -user tom -or -user george           #查找/tmp目录中属于tom或者george这两个用户的文件
find /tmp ! -usr fred                          #查找/tmp目录中不属于fred的文件
 
5.查找并显示文件
find /home/lijiajia/ -name 'elm.cc' -ls        #在目录下查找名为“elm.cc”的文件,并显示这些文件的信息
```

##### 压缩解压 tar

```shell
解包：tar zxvf FileName.tar
打包：tar czvf FileName.tar DirName
```



### 网络操作

##### 查看端口占用

```shell
lsof -i tcp:8080 
netstat -an | grep 3306

netstat -tunlp
```

### 进程 任务操作

##### 搜索进程

```shell
ps -ef|grep treadName //查看程序运行的pid
```

##### 任务切换

```shell
command &   //将进程放在后台执行
ctrl-z      //暂停当前进程 并放入后台
jobs        //查看当前后台任务
bg          //将任务转为后台执行
fg          //将任务调回前台
kill        //杀掉任务
```



### 其他

##### 查看linux发行版本

```shell
lsb_release -a 
```

##### 硬盘挂载操作

```shell
查看所有硬盘信息
sudo fdisk -l
挂载fat32格式U盘
mount -t vfat /dev/sda1 /media 
挂载ntfs硬盘
mount -t ntfs /dev/sda1 /media
卸载u盘
sudo umount -l /media/
```

##### 后台启动 nohup

```
nohup ./test.sh &  挂起服务 退出终端不断开
```



