---
title: 【Linux】工作常用的命令
date: 2024-08-06 23:47:38
tags:
  - Linux
category: 
  - 运维
---



## 历史

查看所有历史命令

```shell
history
```

**搜索包含 `jar` 的历史命令**

执行以下命令仅显示包含 `jar` 的历史命令：

```shell
history | grep jar
```

查找历史删除命令

```shell
history | grep rm
```

## 会话

`screen` 是一个 Linux/Unix 下的终端多路复用器，它允许用户在一个单一终端窗口内启动和控制多个终端会话。在 `screen` 会话中启动的进程，即使用户断开 SSH 连接，进程也能继续运行。

### 创建会话

```shell
# 会话名称
screen -S ts_api
```

在 `screen` 会话中运行命令

```shell
java -jar ts.jar
```

创建

**或者**

```shell
# ts_api 为会话名称
# java -jar /root/soft/test/ts.jar 为会话中执行的命令
# -d 启动一个新的 screen 会话并立即将其置于后台（即分离状态），这样你就不会立即进入这个会话
# -m 强制创建一个新的会话。
# -S 为新创建的会话指定一个名字
screen -dmS ts_api java -jar /root/soft/test/ts.jar
```

### 查看会话列表

```shell
screen -ls
```

返回如下：

```shell
There are screens on:
        998675.ts_api   (Detached)
        4063504.pts-0.ecs-38003 (Detached)
```



### 进入会话

进入就能看到里面的内容，如果在里面运行了jar, 那么进去就能看到日志了

```shell
# 998675.ts_api 从上面的列表中获取
screen -r 998675.ts_api
```

### 退出会话并不终止它

按 `Ctrl+A`，然后按 `D` 键。这将使你退出当前的 screen 会话，而不终止它。

### 停止并删除会话

```shell
screen -S ts_api -X quit
```

## 文件查找

### find

`find` 是 Linux 中的一个强大的命令行工具，用于在文件系统中搜索文件和目录。它可以根据不同的条件（例如名称、大小、类型、时间戳等）进行搜索。下面是 `find` 命令的一些常见用法和选项：

**语法**

```shell
find [路径] [搜索条件] [操作]
```

`[路径]`：指定搜索的起始目录。默认是当前目录。

`[搜索条件]`：定义搜索文件的条件。

`[操作]`：定义对搜索到的文件执行的操作。

**常见选项和示例**

1、按名称搜索文件

```shell
# 在当前目录及其子目录中搜索名为 "filename" 的文件
find . -name "filename"

# 搜索所有扩展名为 ".txt" 的文件
find /path/to/search -name "*.txt"
```

2、按类型搜索

```shell
# 搜索目录
find /path/to/search -type d

# 搜索文件
find /path/to/search -type f
```

3、按大小搜索

```shell
# 搜索大小大于 100MB 的文件
find /path/to/search -size +100M

# 搜索大小小于 1KB 的文件
find /path/to/search -size -1k
```

4、按时间搜索

```shell
# 搜索 7 天内修改过的文件
find /path/to/search -mtime -7

# 搜索 30 天前修改过的文件
find /path/to/search -mtime +30
```

5、按权限搜索

```shell
# 搜索权限为 755 的文件
find /path/to/search -perm 755
```

例子

1、查找当前目录及其子目录中的所有 `.txt` 文件：

```shell
find . -name "*.txt"
```

2、查找 `/home/user` 目录中最近 3 天内修改过的所有文件：

```shell
find /home/user -mtime -3
```

3、查找 `/var/logs` 目录中大于 50MB 的文件并删除它们

```shell
find /var/logs -type f -size +50M -exec rm -f {} \;
```

4、你想要在当前目录及其子目录下搜索所有 Java 文件中的某个关键词，比如 "main"。可以使用以下命令

```shell
find . -name "*.java" | xargs grep -n "main"
```

### grep

`grep` 命令是 Unix 和类 Unix 操作系统（如 Linux）中的一个命令行工具，用于在文件中搜索特定的文本模式或字符串。它的名字是“global regular expression print”的缩写。`grep` 非常强大，支持基本的文本匹配、正则表达式匹配以及各种选项来控制搜索的行为和输出格式。

**语法**

```shell
grep [options] pattern [file...]
```

`pattern`：你要搜索的文本模式或字符串。

`file`：你要在其中搜索的文件（可以是一个或多个）。如果没有指定文件，`grep` 会从标准输入读取。

**常用选项**

- `-i`：忽略大小写。
- `-r` 或 `--recursive`：递归搜索目录中的所有文件。
- `-l`：仅列出包含匹配文本的文件名。
- `-n`：显示匹配行的行号。
- `-v`：显示不包含匹配文本的行。
- `-c`：显示每个文件中匹配的行数。
- `--include`：仅搜索匹配的文件类型。
- `--exclude`：排除匹配的文件类型。

示例

1、使用 `grep` 查找 Java 文件中的关键词

假设你想在当前目录及其子目录下所有 Java 文件中查找关键词 "main"

这个命令会递归搜索当前目录中的所有文件，并仅在扩展名为 `.java` 的文件中查找字符串 "main"

```shell
grep -r "main" . --include="*.java"
```

2、在文件 `file.txt` 中搜索字符串 "hello"：

```shell
grep "hello" file.txt
```

3、递归搜索当前目录及其子目录中的所有文件，查找字符串 "error"：

```shell
grep -r "error" .
```

4、忽略大小写搜索字符串 "pattern" 并显示行号

```shell
grep -in "pattern" file.txt
```

5、在文件 `file.txt` 中搜索字符串 "hello" 并显示匹配行的上下文行：

```shell
# -C 3 会显示匹配行以及其上下各 3 行。
grep -C 3 "hello" file.txt
```

6、仅列出包含匹配文本的文件名

```shell
grep -l "pattern" *.txt
```

7、查找所有扩展名为 `.log` 的文件中包含字符串 "warning" 的行，忽略大小写：

```shell
grep -i "warning" --include="*.log" -r .
```
