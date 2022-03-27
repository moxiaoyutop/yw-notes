# Linux-服务器入侵排查



**Linux服务器疑似被入侵, 如何排查**

1、入侵者可能会删除机器的日志信息，可以查看日志信息是否还存在或者是否被清空

```bash
ll -h /var/log/*
du -sh /var/log/*
```

2、入侵者可能创建一个新的存放用户名及密码文件，可以查看/etc/passwd及/etc/shadow文件

```bash
# 默认两个文件 
[root@localhost ~]#  ll /etc/pass*
-rw-r--r--. 1 root root 846 Jun 14 15:54 /etc/passwd
-rw-r--r--. 1 root root 846 Jun 14 15:54 /etc/passwd-

[root@localhost ~]# ll /etc/sha*
----------. 1 root root 585 Jun 14 15:54 /etc/shadow
----------. 1 root root 590 Jun 14 15:54 /etc/shadow-
```

3、入侵者可能修改用户名及密码文件，可以查看/etc/passwd及/etc/shadow文件内容进行鉴别

```bash
more /etc/passwd
more /etc/shadow
```

4、查看机器最近成功登陆的事件和最后一次不成功的登陆事件，对应日志“/var/log/lastlog”

```bash
[root@localhost ~]# lastlog
Username         Port     From             Latest
root             pts/1    10.10.181.5      Tue Jul 13 13:30:19 +0800 2021
bin                                        **Never logged in**
daemon                                     **Never logged in**
....
```

5、查看机器当前登录的全部用户，对应日志文件“/var/run/utmp”

```bash
[root@localhost ~]# who
root     pts/0        2020-07-13 13:30 (X.X.X.X)
root     pts/1        2020-07-13 13:30 (X.X.X.X)
```

7、查看机器所有用户的连接时间（小时），对应日志文件“/var/log/wtmp”

```
[root@localhost ~]# ac -dp
        root                                23.69
Jun 14  total       23.69
        root                                 6.77
Jul  2  total        6.77
```

8、如果发现机器产生了异常流量，可以使用命令“tcpdump”抓取网络包查看流量情况或者使用工具”iperf”查看流量情况。

9、可以查看/var/log/secure日志文件，尝试发现入侵者的信息

```bash
[root@localhost ~]# cat /var/log/secure | grep -i "accepted password"
Sep 20 12:47:20 hlmcen69n3 sshd[37193]: Accepted password for stone from X.X.X.X port 15898 ssh2
Sep 20 16:17:47 hlmcen69n3 sshd[38206]: Accepted password for stone from X.X.X.X port 9140 ssh2
Sep 20 16:46:00 hlmcen69n3 sshd[38511]: Accepted password for stone from X.X.X.X port 2540 ssh2
Sep 20 16:47:16 hlmcen69n3 sshd[38605]: Accepted password for test01 from X.X.X.X port 10790 ssh2
Sep 20 16:50:04 hlmcen69n3 sshd[38652]: Accepted password for test01 from X.X.X.X port 28956 ssh2
```

10、查询异常进程所对应的执行脚本文件

* top命令查看异常进程对应的PID, 例如PID： 1450
* 在虚拟文件系统目录查找该进程的可执行文件

```bash
[root@localhost ~]# ll /proc/1450/ | grep -i exe
lrwxrwxrwx. 1 root root 0 Sep 15 12:31 exe -> /usr/bin/python
```

11、如果确认机器已经被入侵，重要文件已经被删除，可以尝试找回被删除的文件。

1. 当进程打开了某个文件时，只要该进程保持打开该文件，即使将其删除，它依然存在于磁盘中。这意味着，进程并不知道文件已经被删除，它仍然可以向打开该文件时提供给它的文件描述符进行读取和写入。除了该进程之外，这个文件是不可见的，因为已经删除了其相应的目录索引节点。
2. 在/proc 目录下，其中包含了反映内核和进程树的各种文件。/proc目录挂载的是在内存中所映射的一块区域，所以这些文件和目录并不存在于磁盘中，因此当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息。大多数与 lsof 相关的信息都存储于以进程的 PID 命名的目录中，即 /proc/1234 中包含的是 PID 为 1234 的进程的信息。每个进程目录中存在着各种文件，它们可以使得应用程序简单地了解进程的内存空间、文件描述符列表、指向磁盘上的文件的符号链接和其他系统信息。lsof 程序使用该信息和其他关于内核内部状态的信息来产生其输出。所以lsof 可以显示进程的文件描述符和相关的文件名等信息。也就是我们通过访问进程的文件描述符可以找到该文件的相关信息。
3. 当系统中的某个文件被意外地删除了，只要这个时候系统中还有进程正在访问该文件，那么我们就可以通过lsof从/proc目录下恢复该文件的内容。

例如：

假设入侵者将/var/log/secure文件删除掉了，尝试将/var/log/secure文件恢复的方法可以参考如下：

```bash
# 查看/var/log/secure文件，发现已经没有该文件。
[root@localhost ~]# ll /var/log/secure
cannot access /var/log/secure: No such file or directory


# 使用lsof命令查看当前是否有进程打开/var/log/secure，
[root@localhost ~]# lsof | grep /var/log/secure
rsyslogd   1264      root    4w      REG                8,1  3173904     263917 /var/log/secure (deleted)

# 从上面的信息可以看到 PID 1264（rsyslogd）打开文件的文件描述符为4。同时还可以看到/var/log/secure已经标记为被删除了。因此我们可以在/proc/1264/fd/4（fd下的每个以数字命名的文件表示进程对应的文件描述符）中查看相应的信息，如下
[root@localhost ~]#  tail /proc/1264/fd/4


# 查看/proc/1264/fd/4就可以得到所要恢复的数据。如果可以通过文件描述符查看相应的数据，那么就可以使用I/O重定向将其重定向到文件中
[root@localhost ~]#  cat /proc/1264/fd/4 > /var/log/secure



# 再次查看/var/log/secure，发现该文件已经存在。对于许多应用程序，尤其是日志文件和数据库，这种恢复删除文件的方法非常有用。
[root@localhost ~]#  ll /var/log/secure
```
