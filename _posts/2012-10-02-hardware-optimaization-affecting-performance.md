---
layout: post
title: 两个硬件级优化带来的性能影响
---

{{ page.title }}
===============

计算机的硬件层面的许多优化，比如cpu方面的指令流水线，超标量，多级缓存等技术给系统性能带来极大提升，善用这些技术会给程序执行效能带来极大提升，使用不当则可能带来负面影响，本文要涉及两个与此相关的案例。

#### 指令流水线对程序执行性能的影响。

几个月前酷壳上有篇[文章](http://coolshell.cn/articles/7886.html)提到了排序好的数组在遍历时比起未排序的数组会更快，并给出了这个问题[最初被提到的地方](http://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-an-unsorted-array)（stackoverflow）。

stackoverflow中的范例代码如下：
<pre>
<code>
for (unsigned i = 0; i < 100000; ++i) {
    // primary loop
    for (unsigned j = 0; j < arraySize; ++j) {         
    if (data[j] >= 128)
        sum += data[j];
    }
}
</code>
</pre>
文章中提到其中涉及到的深层次技术是[分支预判](http://en.wikipedia.org/wiki/Branch_predictor)（branch prediction），为了解释分支预判，该文用了扳道工的例子做类比。如果火车要走哪条分支道路能够以很大的概率被正确估计，则火车通过道口的效率将会极大提升。如果估计错误，则火车要退回道口，再次选道，其代价比不预测，换道前先询问的原始方式反而都要高很多。因此提高预测正确的概率对分支预判意义非常重大，具体到示例代码上，排序后的数组正是极大的提高预测正确的概率才带来显著的性能提升。

形而上的说了这么多，但是还有个很重要的问题没弄清楚：如果if语句以及之后的两个分支都是原子的，并且时顺序执行的，排好序的数组和未排序的数组在执行上应该时没有差别的，因此也不应该有效率上的差异。

要理解这个问题，就必须结合cpu内部指令执行的细节。真实情况是在现代cpu的实现中指令的执行既不是原子的，也不是严格顺序执行的。一条指令事实上可能会分成取指令、解码指令、执行指令、写回等多个步骤，交由cpu的各个子部件各自执行，因为各个子部件可以做到独立各自运行，因此就有了cpu流水线这样的先进技术，简单来说就是前一指令稍后环节执行和后续指令的靠前环节可以同时执行，例如一条正处在指令执行阶段，后面的指令就已经可以依次进入取指令，分析指令等环节。

结合示例代码，cpu并不是等分支语句的执行结果出来才决定执行哪个分支，而是通过分支预判来提前决定。这个时候已排序数组的优势就很明显了，排序好的数组预判成功率很高，可以高效率的匹配先进的cpu流水线技术，相反无序数组预判成功率很低，分支语句结果出来之后经常要进行回退，而每次回退代价都很高，此时的流水线加分支预判的技术组合不但没有能够提升语句执行效率，反而成为一种累赘。

#### CPU缓存实现对方阵转置性能的影响。

[原文](http://stackoverflow.com/questions/11413855/why-is-transposing-a-matrix-of-512x512-much-slower-than-transposing-a-matrix-of)同样来自stackoverflow，微博上有人转载。

有人问为什么转置一个512*512的int型矩阵比转置一个513*513的int型矩阵要慢很多，然后就有人解释说都是因为缓存的具体实现，造成了对于行列数为2的整数次幂的矩阵转置操作效率特别低。

文章贴出来的转置代码是这样（有逻辑错误，但不妨碍分析问题）：

<pre>
<code>
void transpose()
{
    for ( int i = 0 ; i < MATSIZE ; i++ )
        for ( int j = 0 ; j < MATSIZE ; j++ )
        {
            int aux = mat[i][j];
            mat[i][j] = mat[j][i];
            mat[j][i] = aux;
        }
}
</code>
</pre>
根据我自己的理解，对场景描述如下，相对原文有所修改：

cpu缓存有32个set，每个set有4行，每行是64字节，每个int类型占4字节。

地址A对应的缓存set为(A/64)%32，set内采取先进先出的方式填满各行。

512*512矩阵每一行正好可以放入32个set，每个set各占一行填满，对于行的枚举来说缓存可以起到作用，但是对于列情况就差很多，同一列所有元素都将落入同一个set，而每个set又只有4行，也就是说最多只有同一列的相邻4个元素能存入set，一旦涉及到新的列就会造成缓存的交换，4相对于512很小，我们可以近似认为对于每一行和对应列的交换都会造成512次的缓存交换，可以认为对一列的遍历缓存命中率趋于0。

而对于513*513的矩阵，其同一列元素正好可以错开而不存在同一列set之中，同时对于一行的遍历来说大约只需占用各个set各一行，因此有足够的空间来存放相应列的元素而不造成换出，此时缓存的命中率将会非常高，表现出来就是时间效率很高。

cpu缓存和流水线技术以及分支预判都是非常先进，高效的底层优化技术，其基本思想都是基于局部性原理：时间局部性原理或者空间局部性原理。虽然计算机科学总体上是符合局部性原理，但任何事务都有两面性，对局部性原理的过分使用会造成特殊情况，极端情况处理效率低下，这就要求我们在使用各种优化技术的同时，不能忽视极端情况，采取相应手段以避开优化技术所带来的负面影响。
