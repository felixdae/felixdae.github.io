---

title: 排序不等式及其证明
---

{{ page.title }}
===============

排序不等式，英文叫rearrangement inequality或者sequence inequality，是这样描述的：

若实数序列$$ a_i $$ ，$$ b_i $$，  $$ 1 \leq i \leq n $$  满足：

$$ a_1 \leq a_2 \cdots \leq a_n $$

$$ b_1 \leq b_2 \cdots \leq b_n $$

则有如下结论：

$$ a_1b_n+\cdots+a_nb_1 \leq a_{\sigma(1)}b_1+\cdots+a_{\sigma(n)}b_n \leq a_1b_1+\cdots+a_nb_n $$

$$\sigma$$是$$[1,\cdots,n]$$到自身上的一个一一映射。

可归纳成：反序和$$\leq$$乱序和$$\leq$$顺序和

这个不等式最初看到应该是在初高中的数学竞赛书上，当时的证明思路是从$$n=2$$进行启发：

$$a_ib_j+a_jb_i \leq a_ib_i+a_jb_j \Longleftrightarrow 0 \leq (a_i-a_j)(b_i-b_j)$$

从而据此认为利用逐步调整的办法可以将乱序和放大至顺序和。中文维基百科的<a href="http://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E4%B8%8D%E7%AD%89%E5%BC%8F" target="_blank">排序不等式</a>条目以及百度google上前几页的中文搜索结果都是这么干的。然而事实上该证明方法暗含一个错误的假定，即认为乱序和中各项都是可两两匹配，但事实上

$$a_1b_2+a_2b_3+a_3b_1 \leq a_1b_1+a_2b_2+a_3b_3$$

就已经无法采用上述所谓的逐步调整法进行操作。

虽然中文维基百科上的证明是错误的，但是英文维基百科上的证明却是正确的，由此也可见中文百科和英文百科质量上的差距。英文百科上该不等式的证明思路是反证法：

只证明右半部分即可，右半部分得证则左半部分是显然的。

所有乱序和中存在一个或者多个最大值，令$$a_{\sigma(1)}b_1+\cdots+a_{\sigma(n)}b_n$$是这样的一个乱序和，并且其还满足一个特点：$$\sigma$$有最大的不动点，即在所有达到最大值的乱序和中，该乱序和有最大的$$j$$使得对于$$[1\cdots j-1]$$满足$$\sigma(i)=i$$，但是$$\sigma(j)\not=j$$。显然我们有$$\sigma(j)>j$$

于是存在$$k>j$$使得$$\sigma(k)=j$$

将$$a_{\sigma(j)}b_j+a_{\sigma(k)}b_k$$放大为$$a_{\sigma(k)}b_j+a_{\sigma(j)}b_k=a_jb_j+a_{\sigma(j)}b_k$$

于是我们得到一个乱序和，要么严格更大，要么有更大的不动点，不论哪种情况都与假设矛盾，于是命题得证。

命题虽然得证，但是还有个疑问：对于三个实数序列显然不存在这样的结论，但是对于三个乃至任意有限多个正实数序列是否这样的结论呢？

