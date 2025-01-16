---
title: LINUX学习
date: 2023-11-09 12:07:09
tags: 学习
---

# Linux

## 安装

* centos7.7镜像下载地址：http://isoredirect.centos.org/centos/7.7.1908/isos/x86_64/
* 安装参考：https://www.cnblogs.com/dreamfreedom/p/11563236.html

## 介绍

* Linux 英文解释为 Linux is not Unix。
* Linux内核最初只是由芬兰人李纳斯·托瓦兹（Linus Torvalds）在赫尔辛基大学上学时出于个人爱好而编写的。
* Linux的发行版说简单点就是将Linux内核与应用软件做一个打包。
* 目前市面上较知名的发行版有：Ubuntu、RedHat、CentOS、Debian、Fedora、SuSE、OpenSUSE、Arch Linux、SolusOS 等。

## 文件结构

```shell
[root@localhost ~]# ls /
bin  dev  home  lost+found  mnt  proc  sbin     srv  tmp  var
boot etc  lib   media       opt  root  selinux  sys  usr
```

* /bin
  
* bin是Binary的缩写, 这个目录存放着最经常使用的命令。
  
* /boot
  
* 这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。
  
* /dev
  
* dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。
  
* /etc
  
* 这个目录用来存放所有的系统管理所需要的配置文件和子目录。
  
* /home
  
* 用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。
  
* /lib
  
* 这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。
  
* /lost+found
  
* 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
  
* /media
  
* linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
  
* /mnt
  
* 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。
  
* /opt
  * 这是给主机额外安装软件所摆放的目录。
  * 比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

* /proc
  * 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
  * 这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器
  ```shell
  echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
  ```

* /root
  
* 该目录为系统管理员，也称作超级权限者的用户主目录。
  
* /sbin
  
* s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
  
* /selinux
  
* 这个目录是Redhat/CentOS所特有的目录，Selinux是一个安全机制，类似于windows的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。
  
* /srv
  
* 该目录存放一些服务启动之后需要提取的数据。
  
* /sys
  * 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。
  * sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。
  * 该文件系统是内核设备树的一个直观反映。
  * 当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。

* /tmp
  
* 这个目录是用来存放一些临时文件的。
  
* /usr
  * 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。
  * /usr/bin
    * 系统用户使用的应用程序。
  * /usr/sbin
    * 超级用户使用的比较高级的管理程序和系统守护程序。
  * /usr/src
    * 内核源代码默认的放置目录。

* /var
  * 这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。
  * 包括各种日志文件。
* 这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在/var/log 目录下，另外mail的预设放置也是在这里。
  
* /etc
  
* 上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。
  

## 文件属性

```she
[root@www /]# ls -l
total 64
dr-xr-xr-x   2 root root 4096 Dec 14  2012 bin
dr-xr-xr-x   4 root root 4096 Apr 19  2012 boot
```

* 第一个字符代表：
  * 当为[ d ]则是目录；
  * 当为[ - ]则是文件；
  * 若是[ l ]则表示为链接文档(link file)；
  * 若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
  * 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

* 从左至右用0-9这些数字来表示
  * 第0位确定文件类型
  * 第1-3位确定属主（该文件的所有者）拥有该文件的权限。
  * 第4-6位确定属组（所有者的同组用户）拥有该文件的权限
  * 第7-9位确定其他用户拥有该文件的权限。
  * 其中，第1、4、7位表示读权限，如果用"r"字符表示，则有读权限，如果用"-"字符表示，则没有读权限；
  * 第2、5、8位表示写权限，如果用"w"字符表示，则有写权限，如果用"-"字符表示没有写权限；
  * 第3、6、9位表示可执行权限，如果用"x"字符表示，则有执行权限，如果用"-"字符表示，则无执行权限。
  * r:4
  * w:2
  * x:1

* 即：`chmod 777 文件名`可赋予所有权限

## 文件操作

```shell
mkdir (创建新目录)
-m ：配置文件的权限喔！直接配置，不需要看默认权限 (umask) 的脸色～
-p ：帮助你直接将所需要的目录(包含上一级目录)递归创建起来！
例如：mkdir -m 711 -p /root/test2

rmdir (删除空的目录)
-p ：连同上一级『空的』目录也一起删除
例如：rmdir -p test1/test2

cp (复制文件或目录)
-a：相当於 -pdr 的意思，至於 pdr 请参考下列说明；(常用)
-d：若来源档为连结档的属性(link file)，则复制连结档属性而非文件本身；
-f：为强制(force)的意思，若目标文件已经存在且无法开启，则移除后再尝试一次；
-i：若目标档(destination)已经存在时，在覆盖时会先询问动作的进行(常用)
-l：进行硬式连结(hard link)的连结档创建，而非复制文件本身；
-p：连同文件的属性一起复制过去，而非使用默认属性(备份常用)；
-r：递归持续复制，用於目录的复制行为；(常用)
-s：复制成为符号连结档 (symbolic link)，亦即『捷径』文件；
-u：若 destination 比 source 旧才升级 destination ！
[root@www ~]# cp [-adfilprsu] 来源档(source) 目标档(destination)
[root@www ~]# cp [options] source1 source2 source3 .... directory

rm (移除文件或目录)
-f ：就是 force 的意思，忽略不存在的文件，不会出现警告信息；
-i ：互动模式，在删除前会询问使用者是否动作
-r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！

mv (移动文件与目录，或修改名称)
-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
-i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！
-u ：若目标文件已经存在，且 source 比较新，才会升级 (update)
[root@www ~]# mv [-fiu] source destination
[root@www ~]# mv [options] source1 source2 source3 .... directory

Linux 文件内容查看
cat  由第一行开始显示文件内容
tac  从最后一行开始显示，可以看出 tac 是 cat 的倒著写！ 全文都显示
nl   显示的时候，顺道输出行号！
more 一页一页的显示文件内容
less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
head 只看头几行
tail 只看尾巴几行

cat [-AbEnTv] 文件
-A ：相当於 -vET 的整合选项，可列出一些特殊字符而不是空白而已；
-b ：列出行号，仅针对非空白行做行号显示，空白行不标行号！
-E ：将结尾的断行字节 $ 显示出来；
-n ：列印出行号，连同空白行也会有行号，与 -b 的选项不同；
-T ：将 [tab] 按键以 ^I 显示出来；
-v ：列出一些看不出来的特殊字符

nl [-bnw] 文件
-b ：指定行号指定的方式，主要有两种：
-b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
-b t ：如果有空行，空的那一行不要列出行号(默认值)；
-n ：列出行号表示的方法，主要有三种：
-n ln ：行号在荧幕的最左方显示；
-n rn ：行号在自己栏位的最右方显示，且不加 0 ；
-n rz ：行号在自己栏位的最右方显示，且加 0 ；
-w ：行号栏位的占用的位数。

more
空白键 (space)：代表向下翻一页；
Enter ：代表向下翻『一行』；
/字串 ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
:f ：立刻显示出档名以及目前显示的行数；
q ：代表立刻离开 more ，不再显示该文件内容。
b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。

less
空白键 ：向下翻动一页；
[pagedown]：向下翻动一页；
[pageup] ：向上翻动一页；
/字串 ：向下搜寻『字串』的功能；
?字串 ：向上搜寻『字串』的功能；
n ：重复前一个搜寻 (与 / 或 ? 有关！)
N ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
q ：离开 less 这个程序；

head [-n number] 文件
-n ：后面接数字，代表显示几行的意思

tail
-n ：后面接数字，代表显示几行的意思
-f ：表示持续侦测后面所接的档名，要等到按下[ctrl]-c才会结束tail的侦测

mkdir dir1 创建一个叫做 'dir1' 的目录' 
mkdir dir1 dir2 同时创建两个目录 
mkdir -p /tmp/dir1/dir2 创建一个目录树 
rm -f file1 删除一个叫做 'file1' 的文件' 
rmdir dir1 删除一个叫做 'dir1' 的目录' 
rm -rf dir1 删除一个叫做 'dir1' 的目录并同时删除其内容 
rm -rf dir1 dir2 同时删除两个目录及它们的内容 
mv dir1 new_dir 重命名/移动 一个目录 
cp file1 file2 复制一个文件 
cp dir/* . 复制一个目录下的所有文件到当前工作目录 
cp -a /tmp/dir1 . 复制一个目录到当前工作目录 
cp -a dir1 dir2 复制一个目录 
ln -s file1 lnk1 创建一个指向文件或目录的软链接 
ln file1 lnk1 创建一个指向文件或目录的物理链接 
```

## 用户管理

```shell
一、Linux系统用户账号的管理
1、添加新的用户账号使用useradd命令
useradd 选项 用户名
-c comment 指定一段注释性描述。
-d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
-g 用户组 指定用户所属的用户组。
-G 用户组，用户组 指定用户所属的附加组。
-s Shell文件 指定用户的登录Shell。
-u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。
2、删除帐号
userdel 选项 用户名
-r，它的作用是把用户的主目录一起删除。
3、修改帐号
usermod 选项 用户名
常用的选项包括-c, -d, -m, -g, -G, -s, -u以及-o等，这些选项的意义与useradd命令中的选项一样，
可以为用户指定新的资源值。
另外，有些系统可以使用选项：-l 新用户名
4、用户口令的管理
passwd 选项 用户名
-l 锁定口令，即禁用账号。
-u 口令解锁。
-d 使账号无口令。
-f 强迫用户下次登录时修改口令。
二、Linux系统用户组的管理
1、增加一个新的用户组使用groupadd命令
groupadd 选项 用户组
-g GID 指定新用户组的组标识号（GID）。
-o 一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。
2、如果要删除一个已有的用户组，使用groupdel命令
groupdel 用户组
3、修改用户组的属性使用groupmod命令
groupmod 选项 用户组
-g GID 为用户组指定新的组标识号。
-o 与-g选项同时使用，用户组的新GID可以与系统已有用户组的GID相同。
-n新用户组 将用户组的名字改为新名字
4、如果一个用户同时属于多个用户组，那么用户可以在用户组之间切换，以便具有其他用户组的权限。
$ newgrp root
```

## 磁盘管理

```shell
Linux磁盘管理好坏直接关系到整个系统的性能问题。
Linux磁盘管理常用三个命令为df、du和fdisk。
df：列出文件系统的整体磁盘使用量
du：检查磁盘空间使用量
fdisk：用于磁盘分区

df
df命令参数功能：检查文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，
目前还剩下多少空间等信息。
语法：
df [-ahikHTm] [目录或文件名]
选项与参数：
-a ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；
-k ：以 KBytes 的容量显示各文件系统；
-m ：以 MBytes 的容量显示各文件系统；
-h ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；
-H ：以 M=1000K 取代 M=1024K 的进位方式；
-T ：显示文件系统类型, 连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；
-i ：不用硬盘容量，而以 inode 的数量来显示

du
Linux du命令是查看使用空间的，但是与df命令不同的是Linux du命令是对文件和目录磁盘使用的空间
的查看，还是和df命令有一些区别的，这里介绍Linux du命令。
语法：
du [-ahskm] 文件或目录名称
选项与参数：
-a ：列出所有的文件与目录容量，因为默认仅统计目录底下的文件量而已。
-h ：以人们较易读的容量格式 (G/M) 显示；
-s ：列出总量而已，而不列出每个各别的目录占用容量；
-S ：不包括子目录下的总计，与 -s 有点差别。
-k ：以 KBytes 列出容量显示；
-m ：以 MBytes 列出容量显示；

fdisk
fdisk 是 Linux 的磁盘分区表操作工具。
语法：
fdisk [-l] 装置名称
选项与参数：
-l ：输出后面接的装置所有的分区内容。若仅有 fdisk -l 时，
则系统将会把整个系统内能够搜寻到的装置的分区均列出来。
```

## vi/vim使用

```shell
基本上 vi/vim 共分为三种模式，分别是命令模式（Command mode），输入模式（Insert mode）
和底线命令模式（Last line mode）。 这三种模式的作用分别是：
命令模式：
用户刚刚启动 vi/vim，便进入了命令模式。
此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。比如我们此时按下i，并不会输入一个字符，
i被当作了一个命令。
以下是常用的几个命令：
i 切换到输入模式，以输入字符。
x 删除当前光标所在处的字符。
: 切换到底线命令模式，以在最底一行输入命令。
若想要编辑文本：启动Vim，进入了命令模式，按下i，切换到输入模式。
命令模式只有一些最基本的命令，因此仍要依靠底线命令模式输入更多命令。
输入模式：
在命令模式下按下i就进入了输入模式。
在输入模式中，可以使用以下按键：
字符按键以及Shift组合，输入字符
ENTER，回车键，换行
BACK SPACE，退格键，删除光标前一个字符
DEL，删除键，删除光标后一个字符
方向键，在文本中移动光标
HOME/END，移动光标到行首/行尾
Page Up/Page Down，上/下翻页
Insert，切换光标为输入/替换模式，光标将变成竖线/下划线
ESC，退出输入模式，切换到命令模式
底线命令模式：
在命令模式下按下:（英文冒号）就进入了底线命令模式。
底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。
在底线命令模式中，基本的命令有（已经省略了冒号）：
q 退出程序
w 保存文件
按ESC键可随时退出底线命令模式。
```

## yum命令

```shell
yum（ Yellow dog Updater, Modified，CentOS中的大黄狗）
是一个在Fedora和RedHat以及SUSE中的Shell前端软件包管理器。
基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，
并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。
yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。
yum 语法
yum [options] [command] [package ...]
options：可选，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），
-q（不显示安装的过程）等等。
command：要进行的操作。
package操作的对象。
yum常用命令
1.列出所有可更新的软件清单命令：yum check-update
2.更新所有软件命令：yum update
3.仅安装指定的软件命令：yum install <package_name>
4.仅更新指定的软件命令：yum update <package_name>
5.列出所有可安裝的软件清单命令：yum list
6.删除软件包命令：yum remove <package_name>
7.查找软件包 命令：yum search <keyword>
8.清除缓存命令:
yum clean packages: 清除缓存目录下的软件包
yum clean headers: 清除缓存目录下的 headers
yum clean oldheaders: 清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :
清除缓存目录下的软件包及旧的headers
```

## bc命令

```shell
bc 命令是任意精度计算器语言，通常在linux下当计算器用。
它类似基本的计算器, 使用这个计算器可以做基本的数学运算。
常用的运算：
+ 加法
- 减法
* 乘法
/ 除法
^ 指数
% 余数
语法
bc(选项)(参数)
选项值
-i：强制进入交互式模式；
-l：定义使用的标准数学库
-w：对POSIX bc的扩展给出警告信息；
-q：不打印正常的GNU bc环境信息；
-v：显示指令版本信息；
-h：显示指令的帮助信息。
参数
文件：指定包含计算任务的文件。

通过管道符
$ echo "15+5" | bc
20

$ bc
2+3
5
```

## &和&&,|和||

```shell
&  表示任务在后台执行，如要在后台运行redis-server,则有  redis-server &
&& 表示前一条命令执行成功时，才执行后一条命令 ，如 echo '1‘ && echo '2'    
| 表示管道，上一条命令的输出，作为下一条命令参数，如 echo 'yes' | wc -l
|| 表示上一条命令执行失败后，才执行下一条命令，如 cat nofile || echo "fail"
```

## tar

```shell
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。
下面的参数是根据需要在压缩或解压档案时可选的。

-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出
下面的参数-f是必须的
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
# tar -cf all.tar *.jpg
这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
# tar -rf all.tar *.gif
这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。

# tar -uf all.tar logo.gif
这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。

# tar -tf all.tar
这条命令是列出all.tar包中所有文件，-t是列出文件的意思

# tar -xf all.tar
这条命令是解出all.tar包中所有文件，-t是解开的意思
压缩
tar -cvf jpg.tar *.jpg        //将目录里所有jpg文件打包成tar.jpg 
tar -czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，
                                生成一个gzip压缩过的包，命名为jpg.tar.gz
tar -cjf jpg.tar.bz2 *.jpg    //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，
                                生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar -cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，
                                生成一个umcompress压缩过的包，命名为jpg.tar.Z
rar a jpg.rar *.jpg           //rar格式的压缩，需要先下载rar for linux
zip jpg.zip *.jpg             //zip格式的压缩，需要先下载zip for linux
解压
tar -xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar -xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
总结
1、*.tar 用 tar -xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar -xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar -xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar -xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压

解压jdk到指定文件夹：
tar -xzvf jdk-8u131-linux-x64.tar.gz -C /usr/local/java
```




