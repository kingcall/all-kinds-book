# HDFS 常用命令

本节主要介绍 HDFS 常用命令，HDFS 命令和 Linux Shell 命令比较相似。每个命令将结合实例进行说明。

## version

查看 Hadoop 版本。
格式：
`version`

示例：
`hdfs dfs version`

## mkdir

创建 HDFS 文件系统目录。
格式：
`mkdir <path>`

示例：
`hdfs dfs -mkdir /user/dir1`

## ls

类似 Linux Shell 的 ls 命令。用它可以列出指定目录下的所有文件。
格式：
`ls <path>`

示例：
`hdfs dfs -ls /user/dir1`

加 -R 参数可以把目录的当前目录下和子目录下的文件都列出来。
`hdfs dfs -ls -R`

## put

把本地文件系统文件和目录拷贝到 HDFS。
格式：
`put <localSrc> <dest>`

localSrc 是本地目录或文件，dest 是 HDFS 目录或文件。

示例：
`hdfs dfs -put /home/dataflair/Desktop/sample /user/dataflair/dir1`

## copyFromLocal

类似 put 命令，但源文件必须是在本地的文件。
格式：

```
copyFromLocal <localsrc> <dest>hdfs dfs -copyFromLocal /home/sample /user/dir1
```

## get

从HDFS下载文件到本地文件系统
格式：`get <src> <localdest>`
示例：`hdfs dfs -get /user/dir1 /home`

把HDFS多个文件合并下载到本地系统的一个文件里面。
`hdfs dfs -getmerge /user/dir1/sample.txt /user/dir2/sample2.txt /home/sample1.txt`

## copyToLocal

类似 get 命令，唯一的区别就是目标路径必须是本地路径。
格式：`copyToLocal <src> <localdest>`
示例：`hdfs dfs -copyToLocal /user/dir1 /home`

## cat

在控制台显示文件内容。
格式：`cat <file-name>`
示例：`hdfs dfs -cat /user/dir1/sample.txt`

## mv

把文件移动到目标路径。
格式：`mv <src> <dest>`
示例：`hdfs dfs -mv /user/dir1/sample.txt /user/dir2`

## cp

把文件或者目录拷贝到指定路径下。
格式：`cp <src> <dest>`
示例：`hdfs dfs -cp /user/dataflair/dir2/sample.txt /user/dataflair/dir1`

## moveFromLocal

类似 put 命令，该命令执行后源文件 local src 被删除，也可以从从键盘读取输入到 hdfs 文件中。
格式：`hadoop fs -moveFromLocal <local src> ... <hdfs dst>`

示例：`hdfs dfs -moveFromLocal /home/dataflair/Desktop/sample /user/dataflair/dir1`

## moveToLocal

把 HDFS 文件或目录拷贝到本地，并删除 HDFS 上的文件或目录。
格式：`hadoop fs -moveToLocal [-crc] <src> <dst>`

如果在 Hadoop Shell 运行这个命令，会提示功能还没实现。
`moveToLocal: Option '-moveToLocal' is not implemented yet.`

## tail

在控制台显示文件最后 1KB 的数据。
格式：
`hdfs dfs -tail [-f] <filename>`

用法：

```
hdfs dfs -tail /user/dataflair/dir2/purchases.txthdfs dfs -tail -f /user/dataflair/dir2/purchases.txt
```

-f 参数可以实时查看文件的变化。

## rm

删除文件和目录，删除目录时需要加 `-r` 参数。
格式：`rm <path>`
示例：

```
$ hdfs dfs -rm /tmp/tmp20190501rm: `/tmp/tmp20190501': Is a directory$ hdfs dfs -rm -r /tmp/tmp2019050119/05/16 10:29:03 INFO fs.TrashPolicyDefault: Moved: 'hdfs://nameservice1/tmp/tmp20190501' to trash at: hdfs://nameservice1/user/ccpgdev/.Trash/Current/tmp/tmp20190501
```

## rmr

格式：`hadoop fs -rmr URI [URI …]`

rm的递归版本。

示例：

```
hadoop fs -rmr /user/hadoop/dirhadoop fs -rmr hdfs://host:port/user/hadoop/dir
```

## expunge

清空回收站
格式：`hdfs dfs -expunge`
示例：`hdfs dfs -expunge`

## chown

改变文件的拥有者。使用 `-R` 将使改变在目录结构下递归进行。命令的使用者必须是超级用户。
格式：
`hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]`

示例：
`hdfs dfs -chown -R dataflair /opt/hadoop/logs`

## chgrp

改变文件所属的组。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。
格式：
`hdfs dfs -chgrp [-R] <NewGroupName> <file or directory name>`

示例：
`hdfs dfs -chgrp [-R] New Group sample`

## chown

改变文件的权限。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。
格式：
`chmod [-R] mode,mode,... <path>...`

示例：
`hdfs dfs -chmod 777 /user/dataflair/dir1/sample`

## setrep

改变一个文件的副本系数。`-R` 选项用于递归改变目录下所有文件的副本系数。
格式：`setrep [-R] [-w] rep <path>`
示例：`hdfs dfs -setrep -w 3 /user/dataflair/dir1`

## du

显示文件的大小
格式：`du <path>`
示例：

```
[dev@pk03 ~]$ hdfs dfs -du /tmp/people0    0     /tmp/people/_SUCCESS343  1029  /tmp/people/part-00000-3eea0d3e-4349-4cc0-90c7-45c423069284-c000.snappy.orc
```

## df

展示 Hadoop 文件系统可用空间。
格式：
`hdfs dfs -df [-h] URI [URI ...]`

示例：
`hdfs dfs -df -h`

## touchz

创建一个0字节的空文件。
格式：
`touchz <path>`

示例：
`hdfs dfs -touchz /user/dataflair/dir2`

## test

用于文件检测操作，如果路径存在，返回 1，如果是目录返回 0。
格式：
`hdfs dfs -test -[ezd] URI`

选项：
-e 检查文件是否存在。如果存在则返回0。
-z 检查文件是否是0字节。如果是则返回0。
-d 如果路径是个目录，则返回1，否则返回0。

示例：

```
hdfs dfs -test -e samplehdfs dfs -test -z samplehdfs dfs -test -d sample
```

## text

将源文件输出为文本格式。允许的格式是 zip 和 TextRecordInputStream。
格式：
`hdfs dfs -text <source>`

示例：
`hdfs dfs -text /user/dataflair/dir1/sample`

## stat

返回指定路径统计信息。
格式：
`hdfs dfs -stat path`

示例：
`hdfs dfs -stat /user/dataflair/dir1`

## appendToFile

追加一个文件到已经存在的文件末尾
格式：
`hadoop fs -appendToFile <localsource> ... <dst>`

示例：
`hadoop fs -appendToFile /home/dataflair/Desktop/sample /user/dataflair/dir1`

## checksum

返回文件的校验和信息。
格式：
`hadoop fs -checksum URI`

示例：
`hadoop fs -checksum /user/dataflair/dir1/sample`

## count

统计一个指定目录下的文件结点数量。
格式：
`hdfs dfs -count [-q] <paths>`

示例：

```
$ hadoop fs -count /testelephant      2      1    108     testelephant
```

第一个数值 2 表示 /testelephant 下的文件夹的个数，
第二个数值 1 表是当前文件夹下文件的个数，
第三个数值 108 表示该文件夹下文件所占的空间大小，这个大小是不计算副本的个数的，单位是字节（byte）。

```
$ hadoop fs -count -q /sunwg     1024     1021    10240    10132     2      1    108    /testelephant
```

在 count 后面增加 -q 选项可以查看当前文件夹的限额使用情况。
第一个数值 1024 ，表示总的文件包括文件夹的限额。
第二个数值 1021 ，表示目前剩余的文件限额，即还可以创建这么多的文件或文件夹。
第三个数值 10240 ，表示当前文件夹空间的限额。
第四个数值 10132 ，表示当前文件夹可用空间的大小，这个限额是会计算多个副本的。
剩下的三个数值与 -count 的结果一样。

## find

找出能匹配上的所有文件。
`-name pattern`不区分大小写，对大小写不敏感。
`-iname pattern`对大小写敏感。
`-print`打印。
`-print0`打印在一行。

格式：
`hadoop fs -find <path> ... <expression> ...`

示例：
`hadoop fs -find /user/dataflair/dir1/ -name sample -print`

## help

查看帮助文档。
格式：
`hadoop fs -help`

示例：
`hadoop fs -help`

## setfattr

为一个文件或者文件夹设置一个扩展属性和值 。
选项参数：
`-b`：除了ACL的文件不移除，他移除所有的扩展属性。所有文件信息都保留给用户，组和其他用户。
`-n name`：显示扩展的属性的名。
`-v value`：显示扩展的属性的值。
`-x name`：移除所有的扩展属性。
`path`：文件或文件夹。

格式：
`hadoop fs -setfattr -n name [-v value] | -x name <path>`

示例：

```
hdfs dfs -setfattr -n user.myAttr -v myValue /user/dataflair/dir2/people.txthdfs dfs -setfattr -n user.noValue /user/dataflair/dir2/people.txthdfs dfs -setfattr -x user.myAttr /user/dataflair/dir2/people.txt
```

## truncate

类似于 linux 的 truncate，用于对文件做截断操作。

格式：
`hadoop fs -truncate [-w] <length> <paths>`

示例：

```
hadoop fs -truncate 55 /user/dataflair/dir2/purchases.txt /user/dataflair/dir1/purchases.txthadoop fs -truncate -w 127 /user/dataflair/dir2/purchases.txt
```

## usage

返回某个命令的帮助文档。
格式：
`hadoop fs -usage command`

示例：

```
$ hdfs dfs -usage mkdirUsage: hadoop fs [generic options] -mkdir [-p] <path> ...
```

