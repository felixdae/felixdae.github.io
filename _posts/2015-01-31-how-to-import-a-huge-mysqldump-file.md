---

title: 如何导入一个巨型的mysqldump文件
---

{{ page.title }}
===============

前几天接到一个任务，要把一个一份mysqldump出来的文件导入数据库，给到的压缩档有接近10G，解压之后达到120G。

之前没干过这种活，以为除了文件大点移动可能比较麻烦点应该没有其他的难度，所以也没多查资料就开搞了。

首先我决定就在我日常用的mbp上来做这个工作，我的电脑是12年买的，i7双核8G内存，心想性能还过得去，唯一的问题是我后来换的ssd容量较小，所以就向老板申请了一个移动硬盘，我设想把原文件挪到这个硬盘上去，然后再在这个硬盘上建一个数据库工作目录。这个过程中碰到几个问题：

1，格式化之后的移动硬盘无法改权限的问题，网上搜了一下，用这个方法解决：
```
usr/bin/sudo /usr/sbin/vsdbutil -a /Volumes/terra
```

2，mysql数据库服务器没法启动，这个部分有上面的权限问题，部分是因为需要对数据库做初始化，使用mysql_install_db脚本初始化一下即可。

解决上面两个问题之后，mysql服务算是起来了，开始导数据。一开始在mysql客户端下使用source命令，这个方法有一个坏处就是不能关闭这个所在的终端窗口。后面在网上找到一个使用重定向的方法。看着硬盘吱吱吱的转，我预估大概一个周末怎么也够导出来了吧。周五晚上开始的，周六一早上我看数据文件夹总大小才不到5G，这个速度下去肯定要坏事，赶紧向老板报告，老板那边说实在不行可以另找外面人来帮忙，可能老板一直忙，最后也没找来帮忙的高人。

这个情况下我不得不开始上网找些资料，网上对于这种场景有一些方案，比如mysql官网上的[这个页面](http://dev.mysql.com/doc/refman/5.5/en/optimizing-innodb-bulk-data-loading.html)。另外要可能要对my.cnf做一些定制，网络上的信息一般说osx上的系统无需使用my.cnf文件，事实上这种场景是需要做一些定制，而且对于osx也是有效的。

另外我想可能问题出在硬盘方面，首先读写都是一个盘，而且还是外接的。所以我做了两件事：

1，对笔记本，另找一个硬盘来作为读的盘，当然也只能用用usb外接。

2，使用raid也许可以加快速度，周六下午就去公司找了台机准备搞，后面发现这台机接口不够，只能再挂一个硬盘，而且还得打开机箱忍受风扇的轰鸣。所以最终也只是用两个硬盘一读一写的方式，也算做个后备方案。这次用percona定制了一个配置文件，但是mysql官网上的方法就没有应用了，因为数据文件实在是太大了，在头尾增加内容都很困难。

3，在京东上下血本买了个500G的ssd。但是等了周日一天都没送货过来。

两台机一起跑，能做的都做了，接下来也就只能等了。到了周日mbp这边还是没起色，导入不到10G，下午正好要去公司附近，顺便看了一下，后备方案反而带来了惊喜，已经看到了有意义的数据。到周一再看又开始出现了另一部分有意义的数据，于是我就彻底把mbp上的导入彻底停了，到了周二终于导完了。导完之后发现一堆表里面有一堆不需要的备份表，这其实提醒了我可以另dump一份数据文件，首先把不需要的表排除在外，其次按表的重要程度分别dump到不同的文件，这样可以选择性的进行导入。

有了一个种子之后，也就可以使用主从同步的方式重建另外的实例了，没必要再倒来倒去巨型原始文件了。做的时候也有几点要注意：

1，在my.cnf中使用replicate-do-table指定要同步的表，多张表可以多次指定。

2，change master指令中的binlog坐标要指向第一个binlog的0位置。

使用这个方法法搭配到货的ssd，发现效果非常好。只一个小时不到就导完了第一部分有意义的数据，之前的方法可是用了一天多的。
