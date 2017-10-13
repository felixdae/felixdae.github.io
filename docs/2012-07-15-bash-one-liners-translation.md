---
layout: post
title: 【自译】bash命令精粹（Bash one-liners）
---

{{ page.title }}
===============


[原文链接](http://www.catonmat.net/blog/bash-one-liners-explained-part-one/)


文件相关操作

## 1，清空一个文件

$ > file

## 2，给文件追加一个字符串

$ echo "foo bar baz" >> file

如果不想追加换行符

$ echo -n "foo bar baz" >> file

## 3，读取文件的第一行并赋给一个变量

$ read -r line < file

或者

$ line=$(head -1 file)

## 4，逐行读取文件

$ while read -r line; do

\# do something with $line

done < file

或者

$ cat file | while IFS= read -r line; do

\# do something with $line

done

## 5，随机的读取文件的一行

$ read -r random_line < <(shuf file)

<(shuf file)会导致的操作有首先打开一个特殊文件/dev/fd/n，然后执行shuf file命令，并把输出连接到/dev/fd/n，并用/dev/fd/n代替<(shuf file)，于是原命令就变成：

$ read -r random_line < /dev/fd/n

下面的两个命令也能完成同样的功能：

$ read -r random_line < <(sort -R file)

$ random_line=$(sort -R file | head -1)

## 6，读取文件的前3个域

$ while read -r field1 field2 field3 throwaway; do

\# do something with $field1, $field2, and $field3

done < file

## 7，获取文件的大小并赋给一个变量

$ size=$(wc -c < file)

## 8，从路径中提取文件名

$ filename=${path\#\#*/}

## 9，从路径中提取目录

$ dirname=${path%/*}

## 10，快速拷贝文件

$ cp /path/to/file{,_copy}

该命令会被展开成

$ cp /path/to/file /path/to/file_copy

字符串相关操作

## 1，生成a-z的字母表

$ echo {a..z}

## 2，去除上例中的空格分割符

$ printf "%c" {a..z}

## 3，定长打印0-9

$ printf "%02d " {0..9}

该语句的效果是

00 01 02 03 04 05 06 07 08 09

## 4，字符串展开成列表

$ echo {w,t,}h{e{n{,ce{,forth}},re{,in,fore,with{,al}}},ither,at}

该语句的效果是

when whence whenceforth where wherein wherefore wherewith wherewithal whither what then thence thenceforth there therein therefore therewith therewithal thither that hen hence henceforth here herein herefore herewith herewithal hither hat

## 5，打印10个相同的字符串

$ echo foo{,,,,,,,,,,}

## 6，连接两个字符串

$ echo "$x$y"

## 7，以特定分割符分隔字符串

$ IFS=- read -r x y z <<< "$str"

## 8，逐个字符处理字符串

$ while IFS= read -rn1 c; do

\# do something with $c

done <<< "$str"

## 9，字符串内替换

$ echo ${str/foo/bar}

替换foo实例

$ echo ${str//foo/bar}

## 10，判断字符串是否匹配特定模式

$ if [[ $file = *.zip ]]; then

\# do something

fi

## 11，判断字符串是否匹配特定正则表达式

$ if [[ $str =~ [0-9]+\.[0-9]+ ]]; then

\# do something

fi

## 12，获取字符串长度

$ echo ${\#str}

## 13，提取子字符串

$ str="hello world"

$ echo ${str:6:3}

将打印wor

${var:offset:length}的含义从字面可解

## 14，字符串转大写

$ declare -u var

$ var="foo bar"

## 15，字符串转小写

$ declare -l var

$ var="FOO BAR"


