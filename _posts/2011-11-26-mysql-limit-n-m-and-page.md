---

title: 数据库"limit n,m"模式强于"page, pagesize"模式
---

{{ page.title }}
===============

mysql的select ... limit n,m语法经常被用来实现分页，比如要拿到第n页的数据，则可以写sql：select ... limit n*pagesiz, pagesize。

但是有一些"limit n,m"能实现的需求，"page,pagesize"模式是无论如何都实现不了的，比如要拿到某大数据集第10条数据以后的所有数据，使用"limit n,m"方法可以写成select .... limit 10,2^32，但是这个功能无法用select ... limit n*pagesiz, pagesize的方法实现。

可能要问这会造成什么问题。一个影响就是按"page, pagesize"思路设计的函数接口的表现力和灵活性要弱于以"limit n,m"语义设计的api。

