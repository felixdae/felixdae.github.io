---

title: 使用openresty测试mysql
---

{{ page.title }}
===============

前几天有人问到使用 mysql 做 id 发生器的 qps 的能达到多少，这方面确实没有数据，近期又在看 openresty，所以就打算用这一套架构来测一下。

系统配置：
mac os x 10.11
core i7双核
8G 内存

mysql 5.6 本机

openresty 1.9.7

最初打算从源码安装openresty，但是依赖蛋疼，尝试了很久居然还不行，后面干脆直接用 brew 安装了，一键搞定。
尝试了一下 openresty mysql 的例子，没出什么大问题，可以工作，要注意一点，lua 代码的搜索路径是 nginx 的安装目录，跟 html 平级，如果代码不是放在这个位置，要在 nginx.conf 指定全路径。

另外用 mysql 做 id 发生器的原理是在某张表 dogs 建一条只有一个字段（主键除外）的记录，每次查出当前值，然后自增该字段，为防止并发，要将这两个语句合在一个事物里面，并且查询语句要加 for update 锁定。

发包工具用的是 Apache 的 ab。

整个测试过程有这么几个变量:
1，ab 的 -n 参数。-c固定为20
2，lua_code_cache
3，nginx 工作进程数

在lua_code_cache为off，nginx 工作进程为1的情况下。
-n 参数为6000时，qps可以达到600+。但是当该参数增为10000时，qps降为300+。

在lua_code_cache为on，nginx 工作进程为2的情况下。
-n 参数为10000时，qps可以达到1700+。但是当该参数增为20000时，ab 会报错，无结果。

在lua_code_cache为on，nginx 工作进程为1的情况下。
-n 参数为10000时，qps可以达到2000+。为6000时，qps 也接近这个数值。

可以看到 lua_code_cache 对性能影响极大，但是进程数增加，性能不升反降，猜测应该是所有程序都跑在一台机器上造成的资源竞争导致。但是正如教程所讲 lua_code_cache 设置为 off 的情况下，改动 lua 代码不需要重启 nginx，可以提高调试效率。

另外大致可以得出：一个独立的 mysql 实例，没有其他表和负载的情况下，只作为 id 发生器，其性能应该至少可以到2000。
