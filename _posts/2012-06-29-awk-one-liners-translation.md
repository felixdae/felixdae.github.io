---

title: 【自译】awk命令荟萃（awk one-liners）
---


{{ page.title }}
===============

[原文](http://www.catonmat.net/blog/awk-one-liners-explained-part-one/)

## 1，加空行

awk '1; { print "" }'

## 2，另一种加空行

awk 'BEGIN { ORS="\n\n" }; 1'

## 3，在非空行之间加空行

awk 'NF { print $0 "\n" }'

## 4，连加两个空行

awk '1; { print "\n" }'

## 5，给每行加编号（各文件之间独立）

awk '{ print FNR "\t" $0 }'

## 6，给每行加编号（各文件之间部不独立）

awk '{ print NR "\t" $0 }'

## 7，另一种加行号的方式

awk '{ printf("%5d : %s\n", NR, $0) }'

## 8，只给非空行加行号

awk 'NF { $0=++a " :" $0 }; { print }'

## 9，算总行数（wc -l）

awk 'END { print NR }'

## 10，每行各列求和

awk '{ s = 0; for (i = 1; i <= NF; i++) s = s+$i; print s }'

## 11，所有行各列求和

awk '{ for (i = 1; i <= NF; i++) s = s+$i }; END { print s+0 }'

## 12，各行每列求绝对值

awk '{ for (i = 1; i <= NF; i++) if ($i < 0) $i = -$i; print }'

## 13，文件总词数

awk '{ total = total + NF }; END { print total+0 }'

## 14，包含Beth的行数

awk '/Beth/ { n++ }; END { print n+0 }'

## 15，打印有最大第一列的行

awk '$1 > max { max=$1; maxline=$0 }; END { print max, maxline }'

awk 'NR == 1 { max = $1; maxline = $0; next; } $1 > max { max=$1; maxline=$0 }; END { print max, maxline }'（升级版）

## 16，在每行行首打印列数

awk '{ print NF ":" $0 } '

## 17，打印每行最后一列

awk '{ print $NF }'

## 18，打印最后一行的最后一列

awk '{ field = $NF }; END { print field }'

awk 'END { print $NF }'

## 19，打印列数大于4的所有行

awk 'NF > 4'

## 20，打印最后列大于4的所有行

awk '$NF > 4'

## 21，转换回车换行（\r\n）成换行（\n）

awk '{ sub(/\r$/,""); print }'

## 22，转换换行（\n）成回车换行（\r\n）

awk '{ sub(/$/,"\r"); print }'

## 23

## 24，用于Windows，忽略

## 25，删除行首空白字符

awk '{ sub(/^[ \t]+/, ""); print }'

## 26，删除行尾空白字符

awk '{ sub(/[ \t]+$/, ""); print }'

## 27，删除行首以及行尾空白字符

awk '{ gsub(/^[ \t]+|[ \t]+$/, ""); print }'

**删除行内所有多余空白字符

awk '{ $1=$1; print }'

## 28，每行行首添加5个空格

awk '{ sub(/^/, " "); print }'

## 29，各行在每行第79字符串处右对齐打印

awk '{ printf "%79s\n", $0 }'

## 30，各行居中打印

awk '{ l=length(); s=int((79-l)/2); printf "%"(s+l)"s\n", $0 }'

## 31，替换第一个foo为bar

awk '{ sub(/foo/,"bar"); print }'

**替换所有foo为bar

awk '{ gsub(/foo/,"bar"); print }'

## 32，将包含baz的各行中的foo替换为bar

awk '/baz/ { gsub(/foo/, "bar") }; { print }'

## 33，将不含baz的各行中的foo替换为bar

awk '!/baz/ { gsub(/foo/, "bar") }; { print }'

## 34，把scarlet，ruby或者puce替换为red

awk '{ gsub(/scarlet|ruby|puce/, "red"); print}'

## 35，反序打印各行

awk '{ a[i++] = $0 } END { for (j=i-1; j>=0;) print a[j--] }'

## 36，连接各行（去掉行尾\，把下一行连接到本行之后）

awk '/\\$/ { sub(/\\$/,""); getline t; print $0 t; next }; 1'

## 37，打印排好序的登录用户名

awk -F ":" '{ print $1 | "sort" }' /etc/passwd

## 38，打印反序的首2列

awk '{ print $2, $1 }' file

## 39，对换1，2列，打印所有行

awk '{ temp = $1; $1 = $2; $2 = temp; print }'

## 40，删除每行第二列

awk '{ $2 = ""; print }'

## 41，每行反序打印

awk '{ for (i=NF; i>0; i--) printf("%s ", $i); printf ("\n") }'

## 42，删除重复的连续行（uniq）

awk 'a != $0; { a = $0 }'

## 43，删除重复行

awk '!a[$0]++'

## 44，以“,”连接连续5行

awk 'ORS=NR%5?",":"\n"'

## 45，打印前10行

awk 'NR < 11'

## 46，打印第一行

awk 'NR > 1 { exit }; 1'

## 47，打印最后2行

awk '{ y=x "\n" $0; x=$0 }; END { print y }'

## 48，打印最后一行

awk '{ rec=$0 } END{ print rec }'

## 49，打印匹配regex的行

awk '/regex/'

## 50，打印不匹配regex的行

awk '!/regex/'

## 51，打印匹配regex的行的上一行

awk '/regex/ { print x }; { x=$0 }'

## 52，打印匹配regex的行的下一行

awk '/regex/ { getline; print }'

## 53，打印匹配AAA，BBB或者CCC的行

awk '/AAA|BBB|CCC/'

## 54，打印以先后顺序包含AAA，BBB，CCC的行

awk '/AAA.*BBB.*CCC/'

## 55，打印长度大于64的行

awk 'length > 64'

## 56，打印长度小于64的行

awk 'length < 64'

## 57，打印从匹配regex的第一行到文件结尾

awk '/regex/,0'

## 58，打印第8到12行

awk 'NR==8,NR==12'

## 59，打印第52行

awk 'NR==52'

awk 'NR==52 { print; exit }'

## 60，打印从匹配Iowa的第一行到匹配Montana的第一行

awk '/Iowa/,/Montana/'

## 61，删除所有空行

awk NF

awk '/./'

