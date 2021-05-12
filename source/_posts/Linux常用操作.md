---
title: Linux常用操作
date: 2021-05-12 17:20:54
update: 2021-05-12 17:20:59
categories: Linux
tags: Linux
---



#### 前言

记录一下工作中用到过的Linux操作。



#### 查看日志

1. tail 

   ```shell
   # 实时监控日志
   tail -f 文件名
   如：tail -f info.log
   
   # 查看日志尾部最后10行日志
   tail -n 10 stdout.log
   ```

2. grep

   ```shell
   grep "要搜索的内容" 文件名
   ```

   

#### 查看文件

1. vim

   ```shell
   vim 文件名
   # 进入编辑模式
   1. i在光标处编辑
   2. A在当前行末尾编辑
   
   # 命令模式
   查找： /  
   下一个： n
   上一个： N
   ```

2. less

   ```shell
   1. less 文件名
   2. less -Nm 文件名 (可以显示 百分比和行号)
   3. 退出 q
   ```



#### 翻页

##### 翻整页

```shell
# 下一页
control + f  (f就是forword)
或者直接 f/空格
# 上一页
control + b  (b就是backward)
或者直接 b
```

##### 翻半页

```shell
# 向下翻半页
control + d （d = down）
或者直接 d
# 向上翻半页
control + u  (u = up)
或者直接 u
```

##### 滚一行

```shell
# 第一种（正在用）
j/k
# 第二种 (记不住)
control + e
control + y
```



#### 压缩

##### zip 压缩/ unzip 解压

```shell
# 将 aaa 目录中的内容压缩成aaa.zip，目录可以是相对路径 -r代表递归
zip -r aaa.zip /Users/didi/Downloads/ChromeDownload/aaa/

# 解压
unzip xxx.zip
```



##### tar压缩

```shell
# 解压
tar -zxvf test.tar.gz
# 压缩
tar -zcvf  test.tar.gz file1 file2
```



##### 7z压缩

它的压缩比率比较高，如果文件数量较大为了节省空间 可以使用这种压缩方式

###### 安装7z

```shell
# mac 安装
brew install p7zip
```



###### 压缩/解压

```shell
# 压缩
7z a -t7z -r test.7z *.mp4 -mx=9
# 参数解释
a：表示add命令，即新建一个压缩文件，该压缩文件存放在当前目录下
-t7z 压缩文件的格式为7z(压缩zip则为-tzip)
-r：表示遍历所有的子目录，每个文件都执行压缩操作，添加到压缩文件中
-mx：表示压缩等级，9级是最高等级。默认等级是5。

# 解压
7z x test.7z
```





#### 查看服务是否启动

##### java服务

```shell
# jps 命令
[root@localhost ~]# jps
31779 xxxApplication
5427 Elasticsearch
24878 Jps
```

##### 其他服务

```shell
# ps aux | grep 项目关键词
[root@localhost ~]# ps aux | grep media-server-standalone
root     12828  0.0  0.0 669156  3860 ?        Sl   1月31   0:25 ./media-server-standalone
root     25030  0.0  0.0 112724   988 pts/1    R+   15:59   0:00 grep --color=auto media-server-standalone

# 或者 ps -ef | grep media-server-standalone
[root@localhost ~]# ps -ef | grep media-server-standalone
root     12828  3788  0 1月31 ?       00:00:25 ./media-server-standalone
root     25462 24837  0 16:04 pts/1    00:00:00 grep --color=auto media-server-standalone

# 可以加 grep -v grep 排除grep本身
ps aux | grep media-server-standalone | grep -v grep

# 使用 lsof -i:8080 查看该端口有没有启起来
[root@localhost ~]# lsof -i:8080
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
java    31779 root  208u  IPv4 12627728      0t0  TCP *:webcache (LISTEN)

# netstat -tunlp|grep 8080
[root@localhost ~]# netstat -tunlp|grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      31779/java 
```

#### kill掉服务

```shell
kill -9 pid
或者
killall -9 media-server-standalone
```

#### nohup后台启动

##### 启动一个go服务

```shell
nohup ./media-server-standalone >/dev/null 2>nohup.out &
```

##### 启动一个脚本

```shell
# 启动看门狗
nohup sh watchdog.sh >> watchdog.log 2>&1 &

# 看门狗
[root@localhost media-server-standalone]# cat watchdog.sh 
while true;
do
    time1=$(date)
    echo $time1
    count=`ps -ef | grep media-server-standalone | grep -v grep`
    if [ "$?" != "0" ];then
        echo  ">>>>no media-server-standalone,run it"
        echo  ">>>>restart media-server-standalone now !"
        nohup ./media-server-standalone >/dev/null 2>nohup.out &
    else
        echo ">>>>media-server-standalone is runing..."
    fi
    sleep 60

```

##### 启动ES

```shell
# 启动 es （需切换到es用户）
./elasticsearch -d
```

##### 启动kibana

```shell
# 启动kibana （需切换到es用户）
nohup ./bin/kibana >> kibana.log 2>&1 &
```

##### nohup用法

```shell
# 只输出错误信息到日志文件
nohup ./program >/dev/null 2>log &
# 什么信息也不要
nohup ./program >/dev/null 2>&1 &
```



#### 磁盘使用情况

##### 查看文件大小

```shell
# df -h
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   50G   23G   28G   46% /
devtmpfs                  16G     0   16G    0% /dev
tmpfs                     16G     0   16G    0% /dev/shm
tmpfs                     16G  234M   16G    2% /run
tmpfs                     16G     0   16G    0% /sys/fs/cgroup
/dev/md126p2            1014M  164M  851M   17% /boot
/dev/md126p1             200M  9.8M  191M    5% /boot/efi
/dev/mapper/centos-home  3.4T  233G  3.2T    7% /home
tmpfs                    3.2G     0  3.2G    0% /run/user/0


# du -h --max-depth=1
[root@localhost ~]# du -h --max-depth=1
12M	./.cache
4.0K	./.dbus
0	./.config
4.3G	./work
196K	./firmware
112K	./data1
4.3G	.

# du -sh *
[root@localhost output]# du -sh *
16K	bin
20K	conf
0	Dockerfile
4.0K	emptystdlog.sh
28M	gc.log.2021-01-13_18_17_58
61M	gc.log.2021-02-18_14_43_46
75M	lib
4.0G	stdout.log
```



##### 在Linux上用U盘或移动硬盘拷贝数据

1. **通过 fdisk -l  找到自己的U盘**

   ```shell
   [root@localhost ~]# fdisk -l
   WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.
   
   磁盘 /dev/sda：4000.8 GB, 4000787030016 字节，7814037168 个扇区
   Units = 扇区 of 1 * 512 = 512 bytes
   扇区大小(逻辑/物理)：512 字节 / 512 字节
   I/O 大小(最小/最佳)：512 字节 / 512 字节
   磁盘标签类型：gpt
   Disk identifier: C45A17BF-A83E-422B-A363-A1A50D6D29F8
   
   
   #         Start          End    Size  Type            Name
    1         2048       411647    200M  EFI System      EFI System Partition
    2       411648      2508799      1G  Microsoft basic 
    3      2508800   7423322111    3.5T  Linux LVM       
   WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.
   
   磁盘 /dev/sdb：4000.8 GB, 4000787030016 字节，7814037168 个扇区
   Units = 扇区 of 1 * 512 = 512 bytes
   扇区大小(逻辑/物理)：512 字节 / 512 字节
   I/O 大小(最小/最佳)：512 字节 / 512 字节
   磁盘标签类型：gpt
   Disk identifier: C45A17BF-A83E-422B-A363-A1A50D6D29F8
   ```

   **比如找到自己的U盘：/dev/sdc1**

2. **加载USB模块**

   ```shell
   modprobe usb-storage
   ```

3. 在 目录 /mnt 下创建 文件夹 usb_disk

   ```shell
   mkdir /mnt/usb_disk
   ```

4. 挂载U盘

   ```shell
   # 方式1
   mount /dev/sdc1 /mnt/usb_disk/
   # 方式2
   mount -t ntfs /dev/sdc1 /mnt/usb_disk/
   ```

   如果有问题，需要下载 `ntfs-3g`，然后执行 `mount  -t ntfs-3g /dev/sdc1 /mnt/usb_disk/`

   ```shell
   # 下载
   wget https://tuxera.com/opensource/ntfs-3g_ntfsprogs-2017.3.23.tgz
   # 解压
   tar xf ntfs-3g_ntfsprogs-2017.3.23.tgz
   # 目录位置
   /usr/local/ntfs-3g_ntfsprogs-2017.3.23
   # 编译安装
   cd ntfs-3g_ntfsprogs-2017.3.23
   ./configure && echo $?
   make && echo $?
   make install && echo $?
   ```

5. 卸载U盘

   ```shell
   umount /mnt/usb_disk/
   ```

   

#### scp上传下载

```shell
# 本地上传文件到服务器
scp test.7z root@192.168.41.211:/root/work

# 服务的文件 传到本地
scp root@192.168.41.211:/root/work/test.7z ./
```



#### 防火墙

```shell
# 查看 开通的端口
firewall-cmd --zone=public --list-ports
# 开通某个端口（移除某个端口）
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
# 重启防火墙
firewall-cmd --reload
```



#### 修改某个文件的用户和用户组

```shell
# 对elk-es-cluster_server.json文件 设置用户和用户组分别为es和es
chown es:es elk-es-cluster_server.json
# 如果 需要递归设置 加 -R 参数
```



#### 查找指令

```shell
whereis java
find / -name java
```

