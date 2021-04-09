# Linux常用命令

[TOC]

## 一.系统管理

#### 1.配置ssh

##### （1）生成SSH密匙

```bash
$ sh-keygen -t rsa [全默认,回车]
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@120.77.251.152 [回车,输入密码]
```

> cat ~/.ssh/id_rsa.pub 输出公钥，可添加到Github验证

##### （2）创建别名

用于快捷登录。Windows下建议使用XShell。

```bash
$ vim ~/.ssh/config

    Host aliyun

    HostName 120.77.251.152

    User root

$ sudo chmod 600 /home/zhang/.ssh/config
```

#### 2. 查询端口占用

```bash
$ netstat -apn |grep 8088
```

#### 3. 向服务器传输文件
```bash
$ scp ./wechat.py root@120.77.251.152:/root/Desktop/ #-r 传递文件夹
```

#### 4. Linux误操作嘟嘟声去掉方法

`sudo vi /etc/inputrc` ，找到 setbell-style none 把前面的注释去掉，保存退出，然后重启系统.

#### 5. 配置apt清华源

`/etc/apt/sources.list`备份后修改为以下内容。

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```


## 二.日常使用

#### 1. 建立软连接（快捷方式）

```bash
$ ln -s [文件名] [软连接名]
$ ln -s /media/zhangweizhen/新加卷/Tutorials
```

#### 2. 网络下载

curl，-L更随重定向

```bash
$ curl -LO http://download.tensorflow.org/example_images/flower_photos.tgz
```

wget，-c：断点续传，-P：下载到指定文件夹。

```bash
$ wget -c http://mattmahoney.net/dc/enwik9.zip -P ./data
```

#### 3. 层级查找文件

find，-name按文件名查找

```bash
$ find /etc -name profile
```

#### 4. 解压文件
```bash
$ unzip -O cp936 xxx.zip
```

#### 5. 查找命令所在位置
```bash
$ which python3
```

#### 6.重定向

##### (1) 输入/覆盖

```bash
$ echo "text" > newFile.txt
```

##### (2)追加

```bash
$ cat ~/.bash_profile >> ~/.zshrc
```

#### 7.文本文件编码转换

```bash
iconv -f GBK -t UTF-8 [GBK文件] > [UTF-8文件]
```

## 三.开发环境

#### 1.Vim

| 快捷键   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| ^        | 跳到行首                                                     |
| g_       | 跳到行尾                                                     |
| e        | 跳到写个单词                                                 |
| %        | 匹配括号移动，包括 (, {, [. （注：你需要把光标先移到括号上） |
| * 和 #   | 匹配光标当前所在的单词，移动光标到下一个（或上一个）匹配单词（*是下一个，#是上一个） |
| yy       | 复制当前行                                                   |
| p        | 粘贴                                                         |
| gg       | 跳到文件开头                                                 |
| G        | 跳到文件末尾                                                 |
| dt"      | 删除所有的内容，直到遇到双引号--"                            |
| cw       | 删除当前单词并进入编辑模式                                   |
| a        | 在光标后插入                                                 |
| o        | 在当前行后插入一个新行                                       |
| O        | 在当前行前插入一个新行                                       |
| /pattern | 搜索 pattern 的字符串（注：如果搜索出多个匹配，可按n键到下一个） |
| u        | undo                                                         |
| <C-r>    | redo                                                         |
| :wq      | 保存并退出                                                   |
| :100     | 跳到第100行                                                  |



#### 2.创建数据库

```
create database ihome default charset=utf8；
```

#### 3.Nginx
```
./nginx //启动

./nginx -s quit //终止
```

#### 4.xxd

使用16进制查看文本文件

## 四.磁盘管理

#### 1. 查看磁盘空间
```bash
df -h
```

#### 2.查看目录大小

##### (1)查看/etc目录总大小

```bash
$ du -sh /etc
```

##### (2)查看/etc下所有目录大小

```bash
$ du -h /etc 
```

#### 3.挂载命令

显示挂载的设备
```bash
$ mount -l
```

挂载
```bash
$ mkdir /mnt/cdrom

$ mount /dev/cdrom /mnt/cdrom

$ mount -o remount,ew /
```

卸载
```bash
$ unmount /mnt/cdrom 
```

> 卸载时可以指定挂载点或设备文件名

查看U盘设备文件名并挂载
```bash
$ fdisk -l

$ mount -t vfat /dev/sdb1 /mnt/usb
```

> systs,proc是内存的挂载点，tempfs、devpts等是临时挂载点

#### 4.fdisk分区命令

##### (1)分区要点

- 主分区最多4个
- 扩展分区最多1个，且扩展分区不能格式化也不能存储数据只能用于包含逻辑分区
- 逻辑分区分区号从5开始
- 主分区+扩展分区最多4个
- 一般是一个主分区(分区号为1)和一个扩展分区(分区号为2)

查看硬盘和分区情况
```bash
$ fdisk -l
```

> fdisk命令不能知道硬盘是否被挂载，要知道是否被挂载要使用`df`或`mount`命令

##### (2) 分区命令

```bash
$ fdisk /dev/sdb
```

##### (3) 交互指令

- n：新建分区
- l：显示已知的文件系统类型，82位Linux swap分区，83为Linux分区
- m：显示帮助
- d：删除分区
- w：保存退出
- q：不保存退出
- p：显示分区列表

> 可能需要按Ctrl+Backspace删除

##### (4) 分区过程

```
命令(输入 m 获取帮助)：n

Partition type:

   p   primary (1 primary, 0 extended, 3 free)

   e   extended

Select (default p): p

分区号 (2-4，默认 2)：2

起始 扇区 (83884032-83886079，默认为 83884032)：

将使用默认值 83884032

Last 扇区, +扇区 or +size{K,M,G} (83884032-83886079，默认为 83886079)：+10G

Last 扇区, +扇区 or +size{K,M,G} (83884032-83886079，默认为 83886079)： 
```

> 默认按扇区分配不符合习惯，可以使用`+10G`这样的方式来指定分区大小

重新读取分区表(分区之后最好执行一遍)
```bash
$ partprobe
```

##### (5) 格式化分区

```bash
$ mkfs -t ext4 /dev/sdb1
```

##### (6)新建空目录挂载分区

```bash
$ mkdir /disk1

$ mkdir /disk2

$ mount /dev/sdb1 /disk1

$ mount /dev/sdb5 /disk2
```

#### 5.自动挂载

##### (1)`/etc/fstab`文件

自动挂载/dev/sdb1到/disk1目录，每天自动备份，fsck优先级为2
```bash
/dev/sdb1 /disk1 ext4 defaults 1 2
```

按照/etc/fstab文件的内容自动挂载，如有错误会报错
```bash
$ mount -a
```