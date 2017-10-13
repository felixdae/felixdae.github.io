---
title: Bloom filter的参数选择
---

{{ page.title }}
===============

#### Bloom filter 是一种类似集合的数据结构，主要用来查询某元素是否在集合中。一般提供两种操作：check 和 add，check 用来查询某一 key 是否存在，add 用来将新 key 加入集合。

#### Bloom filter 的原理是将一个 key 用 k 个独立的 hash 函数映射到 k 个 bit 位，如果这 k 个位都为1，这说明该 key 在集合中。如果只要有一个位为0，则说明 key 不在集合中。

#### 很显然，如果 Bloom filter 说某 key 不在集合中，则该 key 确实不在集合中；相反如果说某 key 在集合中，则有一定的概率属于误报。在实际应用中，在确定了目标容量的前提下，可以通过选择合适的 hash 函数个数 k ，让这个概率尽可能低。

#### 假定我们的物理存储有 m 个 bit，使用了 k 个 hash 函数，而且已经存储了 n 个 key，则此时再新增一个 key，发生冲突的概率是：
![]({{ site.img_url }}/14741682059426.png)

#### 在设计 bloom filter 时，如果 m 和 n 确定，也就是物理存储大小和目标容量已经确定的情况下，要让冲突概率尽可能小，也就是让上面的式子尽可能小，该如何选择最优的 k ？可以通过求导的方式来求解。令 \\( t = e^{-kn/m} \\) ，可解得当 \\( t=1/2 \\) 时，也就是 \\( k=m \cdot ln2/n \\) 时，可使得冲突率最低。

#### 有时候我们可能预设了一个目标冲突率，这个时候选择合适的 k 则可以使得系统的容量——也就是 n 尽可能大。

\\[ (1-e^{-kn/m})^k = p => \\]
\\[ n=ln(1-p^{1/k})\cdot(-m)/k \\]

#### 如上式 m 和 p 已经确定，n 是 k 的函数，要求得 n 最大时的 k ，同样令 \\(  t=p^{1/k} \\) 并用求导的方式，可解得当 \\( t=1/2 \\)时，也就是 \\( k=-lnp/ln2 \\)时可使得 n 最大。**这里求得的 k 正好是概率为 p 的事件的信息量的计算公式，不知是巧合还是还是这个问题与信息论有关联，是否从信息论的角度可以轻松的给出以上结论？**
