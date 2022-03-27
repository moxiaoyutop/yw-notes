# awk|grep|sed|find



## awk

### 1、awk 其他用法

```bash
awk '!S[$0]++' filename     # 去除文件重复的行
awk 'S[$0]++' filename      # 显示文件重复的行
awk '!S[$1]++' filename     # 去除第一列相同的行
awk '!S[$2]++' filename     # 去除第二列相同的行

awk '{S[$0]++}END{ for(i in S) print i"\t\t"S[i]}' filename   # 去重，并统计每一行出现的次数
awk '{S[$2]++}END{ for(i in S) print i"\t\t"S[i]}' filename   # 去重，并统计第二列字符出现的次数
```

### 2、使用awk匹配用法

```bash
awk 'NR < 5' filename                                                            # 行号小于5
awk 'NR==1,NR==4 {print}' filename                                               # 打印出来1和4的行号
awk '/linux/' filename                                                           # 包含linux文本的行
awk '!/linux/'                                                                   # 不包含linux文本的行
awk '/stat/, /end/' filename                                                     # 打印处于start 和 end 之间的文本
```

### 3、for 在 awk 中用法

```bash
netstat -n | awk '/^tcp/ {S[$NF]++} END {for(i in S) print i"\t\t"S[i]}'         # 查看并发请求数及其 TCP 连接状态
netstat -nat |awk '{print $6}'|sort|uniq -c|sort -rn                             # 查看tcp连接状态
```

### 4、if 在 awk 中用法

```bash
ps -eal | awk '{ if ($2 == "Z") {print $4}}' | kill -9                           # 清除僵死进程
chkconfig --list | awk '{if ($5=="3:on") print $1}'                              # 查看系统自启动的服务
```

### 5、BEGIN END 用法

```bash
awk 'BEGIN{num = 0 ;print "begin";} {sum += $1;} END {print "=="; print sum }'   # 计算第二行的和
awk '{sum += $2} END {print sum }'                                               # 计算第二行的和
awk ' END {print NR}' file                                                       # 统计文件的行数
find / -name *.jpg -exec du -sh {} \;|awk '{print $1}'|awk '{a+=$1}END{print a}' # 统计所有的 jpg 的文件的大小
```

### 6、变量在 awk 中用法

```bash
var=1000 ; echo | awk '{print vara}' vara=$var                                   # 传递外部变量
```

### 7、合并文本两行为一行

```bash
awk '{tmp=$0;getline;print tmp"\t"$0}' test.txt
```



## grep

> \-o 只输出匹配的文本行 VS -v 只输出没有匹配的文本行 -c 统计文件中包含文本的次数 -n 打印匹配的行号 -i 搜索时忽略大小写 -l 只打印文件名 -w 精准匹配 -R 递归 -e 多条件 -A ：后跟一个数字（有无空格都可以），例如 A2则表示打印符合要求的行以及下面两行 -B ：后跟一个数字，例如 B2 则表示打印符合要求的行以及上面两行 -C ：后跟一个数字，例如 C2 则表示打印符合要求的行以及上下各两行

### 1、常用命令

```bash
grep -A2 'str' filename                # 把包含 str 的行以及这行下面的两行都打印出
grep -B2 'str' filename                # 把包含 str 的行以及这行上面的两行都打印出
grep -C2 'str' filename                # 把包含 str 的行以及这行上面和下面的各两行都打印出。
grep -n 'str' filename                 # 过滤出带有某个关键词的行并输出行号 
grep -nv 'str' filename                # 过滤不带有某个关键词的行，并输出行号
grep '[0-9]' filename                  # 过滤出所有包含数字的行 
grep -v '[0-9]' filename               # 过滤出所有不包含数字的行 
grep -v '^#' filename                  # 把所有以 # 开头的行去除 
grep -v -e '^#' -e '^$'  filename      # 去除所有空行和以 ‘#' 开头的行 
grep '^[^a-zA-Z]' filename             # 去除所有以字母开头的行
grep '[^0-9a-zA-Z]' filename           # 去除所有数字以及大小写字母开头的行
grep 'str*' filename                   # 模糊匹配包含 str 的字符

egrep 'str1+|str2+' filename           # 匹配一个或一个以上前面的字符 
egrep 'o?|oo?' filename                # 匹配零个或一个前面的字符 
egrep 'str1|str2|str3' filename        # 匹配多个条件， 类似 grep -e
```

***

例如：文件目录

```bash
[root@k8s-node02 ~]# tree
├── dir1
│   └── test1.txt
└── dir2
    └── test2.txt
```

### 2、递归**精准**查询当前路径下包含 test 的文件及行号

```bash
[root@k8s-node02 ~]# grep -R -n  -w test  .     || grep -rn -w test .                
./dir1/test1.txt:4:test
./dir2/test2.txt:4:test
```

### 3、递归模糊查询当前路径下包含 test 的文件名

```bash
[root@k8s-node02 ~]# grep -R -l  test . 
./dir1/test1.txt
./dir2/test2.txt
```

### 4、递归模糊查询当前路径下包含 test 的文件及行号

```bash
[root@k8s-node02 ~]# grep -R -n  test .              
./dir1/test1.txt:1:test1-1
./dir1/test1.txt:2:test1-2
./dir1/test1.txt:3:test1
./dir1/test1.txt:4:test
./dir2/test2.txt:1:test2-1
./dir2/test2.txt:2:test2-2
./dir2/test2.txt:3:test2
./dir2/test2.txt:4:test
```

### 5、匹配多个条件

```bash
[root@k8s-node02 ~]# grep -R -n  -e test1-1 -e test2-1  .
./dir1/test1.txt:1:test1-1
./dir2/test2.txt:1:test2-1
```

### 6、统计文件中 test 出现的次数

```bash
[root@k8s-node02 ~]#  grep -c "test" ./dir1/test1.txt   
4
```



## sed

### 1、常见参数



* n 取消默认输出(输出所有文本内容)，-n只显示处理过的行
* \-i 直接操作文件
* \-f 使用sed脚本
* \-e 连续编辑
* p 打印匹配的内容，通常与-n一起使用
* a 追加 < 插入当前行的后面一行 >
* i 插入 < 插入当前行的前面一行 >
* c 更改
* d 删除
* s 替换
* p 打印
* \= 打印匹配的行号
* n 读取下一行
* r,w 读和写

### 2、删除操作，！为取反操作

```bash
sed '2d' filename             # 删除第2行
sed '2!d' filename            # 删除第2行以外的所有行
sed '1,2d' filename           # 删除第1、2行
sed '2,+1d' filename          # 删除第2行及后面的一行
sed '1~3d' filename           # 从第1行开始，每隔3行删除一行
sed '$'d filename             # 删除最后一行
sed '/^$/d' filename          # 删除空行
sed '/aaa/d' filename         # 删除匹配 aaa 的行
sed '/aaa\|bbb/' filename     # 删除匹配 aaa 或者 bbb 的行
sed '1,10{/aa/d}' filename    # 删除 1~10 行匹配 aa 的行
sed '/aaa/,$d' filename       # 删除匹配 aaa 行到最后一行 
sed '/^#/d;/^$/d' filename    # 删除注释行和空行
sed -i '/[:blank:]*#/d' filename # 删除一个或多个空格加 # 号的行
```

### 3、插入新行

```bash
# a:插入当前行的后面一行， i:插入当前行的前面一行,  c:更改行
sed 'atest' filename          # 在每一行后面插入 test 行
sed '2atest' filename         # 在第2行后面插入 test 行
sed '2!atest' filename        # 在除了第2行的每一行后面插入 test 行
sed '/hello/atest' filename   # 在匹配行后面插入 test
sed '$atest' filename         # 在最后一行后面插入 test
```

### 4、替换操作

```bash
sed 's/aaa/bbb/' filename        # 替换所有行中第一个 aaa 为 bbb
sed 's/aaa/bbb/2' filename       # 替换所有行中第二个 aaa 为 bbb
sed 's/aaa/bbb/g' filename       # 替换所有的 aaa 为 bbb 
sed '1,10s/aaa/bbb/g' filename   # 替换第1行~第10行所有的 aaa 为 bbb
sed 's/^[0-9]/(&)/' filename     # 将数字加上一个(), &为匹配到的内容
sed "/ccc/{s/aaa/bbb/g;q}"       # 匹配ccc,并且把含有ccc的行中 aaa 都替换成 bbb, {}里可以执行多个命令，用;隔开即可,q是退出
```

### 5、多个 sed 命令组合

```bash
sed -e "2d" -e "s/last/new/"     # 删除第二行，并且匹配把last替换成new
```

### 6、引用变量

```bash
sed -i "s/$old_str/$new_str/" filename
sed -i s/$old_str/$new_str/ filename
sed -i 's#'''$old_str'''#'''$new_str'''#g' file     # 当变量中存在特殊字符/,将/改为#
```

### 7、合并文本两行为一行

```bash
sed -n '{N;s/\n/\t/p}' test.txt
```

### 8、过滤多了200b字符文件

**vim**查看文件，发现多了<200b>字符，使用/200b搜索匹配不上；

```
grep 200b 也匹配不上
```

处理方法见： [http://superuser.com/questions/207207/how-can-i-delete-u200b-zero-width-space-using-sed](http://superuser.com/questions/207207/how-can-i-delete-u200b-zero-width-space-using-sed)

```
sed 's/\xe2\x80\x8b//g' inputfile
```

\


## find

{% hint style="info" %}
语法： find \[路径] \[参数]\
注：如果不输入路径，查询当前目录！
{% endhint %}

* \-perm <权限数值>
* \-name 文件名字
* \-iname 忽略文件名的大小写，匹配所有大小写字母
* \-type f文件，d目录，l连接文件，b块设备，c串行端口设备
* \-size 通过文件大小查找
* \-inum 查找 inode
* \-user 指定属主，也可以使用 uid
* \-group 指定用户组，也可以使用 gid
* \-not 查找不满足条件的文件，用在特定的条件之前
* \-o 或者
* \-a 并且
* \-mindepth 指定目录的开始深度
* \-maxdepth 指定目录的最大深度
* \-\*time mtime 创建或更改时间；atime 访问时间；ctime文件inode号被修改。
* \-\*min mmin ±n，大于小于 n 分钟
* \-mtime +365 创建或更改时间，大于365天的
* \-mtime -10 创建或更改时间，小于10天
* \-atime +365 访问或读取时间，大于365天
* \-atime -10 访问或读取时间，小于10天

### 1、根据文件或者正则表达式进行匹配

```bash
find /tmp -name "aming*"                       # 查找 /tmp 目录下名字为 aming开头的所有文件。
find /tmp -iname abcde                         # 查找 /tmp 目录下包含abcde字母的文件，不区分大小
find /tmp -name "a*" -a -name "*c" -type f     # 搜索 /tmp 目录下以 a 开头并且以 c 结尾的文件, 类似的还有：-a 且  ，  -not 不满足

find /tmp -path "*local"                       # 匹配文件路径或者文件
find /tmp -iregex ".*\(\.txt\|\.pdf\)$"        # 基于正则表达式匹配文件路径
```

### 2、否定参数

```bash
find /tmp -type f ! -mtime -365                # 列出tmp目录下一年内都没有改变的文件
```

### 3、根据文件类型进行搜索

```bash
find /tmp -type f -name ".*"                   # 在 /tmp 目录下查找所有的文件。
```

### 4、基于目录深度搜索

```bash
find /tmp -maxdepth 3 -name "*.txt"            # 搜索出最大目录层级为3的所有txt文件
```

### 5、根据文件时间戳进行搜索

```bash
find /tmp -type f  -mtime -365                 # 搜索 tmp 目录下，修改时间一年内的文件
find /tmp -not -type f -mtime -365             # 搜索目录下，修改时间一年内，不是文件的其他类型
find /tmp -type f -atime +10                   # 找出访问时间超过10分钟的所有文件
find /tmp -type f -mtime +10                   # 找出修改时间超过10分钟的所欲文件
```

### 6、根据文件大小进行匹配

```bash
find /tmp -name "*.txt" -size -10k             # 搜索小于10KB的文件
find /tmp -name "*.txt" -size +10G             # 搜索大于10G的文件
```

### 7、删除匹配文件

```bash
find /tmp -name "*.txt" -size +10G -delete     # 删除目录下所有大于10G的 txt 文件
```

### 8、根据文件权限进行匹配

```bash
find /tmp -type f perm 644                     # 找出当前目录下权限为644的文件
find /tmp -type f -user test                   # 找出当前目录用户test拥有的所有文件
find /tmp -type f -group test                  # 找出当前用户组test拥有的所有文
```

### 9、结合 -exec 使用

```bash
find /tmp -name  "*.txt" -user test -exec chown root {} \;
```

### 10、搜索但跳过指定的目录

```bash
find / -path "./root" -prune -o -name "*local"  # 跳过 root 目录查找 *local* 文件
```

### 11、搜索属于空文件，空目录，

```bash
find / -empth                                  # 搜索属于空文件，空目录，
```
