---
layout: post
title: How many errors are left to find?
---

{{ page.title }}
===============

（转自http://www.johndcook.com/blog/2010/07/13/lincoln-index/）

How many errors are left to find?
by John on July 13, 2010

There’s a simple statistic called the Lincoln Index that lets you estimate the total number of errors based on the number of errors found. I’ll explain what the Lincoln Index is, why it works, give some code for playing with it, and discuss how it applies to software testing.

What is the Lincoln Index?

Suppose you have a tester who finds 20 bugs in your program. You want to estimate how many bugs are really in the program. You know there are at least 20 bugs, and if you have supreme confidence in your tester, you may suppose there are around 20 bugs. But maybe your tester isn’t very good. Maybe there are hundreds of bugs. How can you have any idea how many bugs there are? There’s no way to know with one tester. But if you have two testers, you can get a good idea, even if you don’t know how skilled the testers are.

Suppose two testers independently search for bugs. Let E1 be the number of errors the first tester finds and E2 the number of errors the second tester finds. Let S be the number of errors both testers find. The Lincoln Index estimates the total number of errors as E1 E2/S. You can find historical background on the Lincoln Index here.

How does the index work?

Suppose there are n bugs and the two testers find bugs with probability p1 and p2 respectively. You’d expect the two testers to find around np1 and np2 bugs. If you assume the probabilities of each tester finding a bug are independent, you’d expect the testers to find around np1 p2 bugs in common. That says E1*E2/S would be around

(n2 p1 p2) / (n p1 p2) = n.

The probabilities of each tester finding a bug cancel out leaving only n, the total number of bugs.

Simulation code

Here’s some Python code for simulating estimates using the Lincoln Index.

<pre>
<code>
from random import random
 
def find_error(p):
    "Find an error with probability p"
    if random() < p:
        return 1
    return 0
 
def simulate(true_error_count, p1, p2, reps=10000):
    """Simulate Lincoln's method for estimating errors
    given the true number of errors, each person's probability
    of finding an error, and the number of simulations to run."""
    estimation_error_sum = 0
    for rep in xrange(reps):
        caught1 = 0
        caught2 = 0
        caught_both = 0
        for error in xrange(true_error_count):
            found1 = find_error(p1)
            found2 = find_error(p2)
            caught1 += found1
            caught2 += found2
            caught_both += found1*found2
        estimate = caught1*caught2 / float(caught_both)
        estimation_error_sum += abs(estimate - true_error_count)
    return estimation_error_sum / float(reps)
</code>
</pre>
I used this to simulate the case of two testers, one with a 30% chance of finding a bug and the other with a 40% chance, and a total of 100 bugs. I simulated the Lincoln Index 1,000 times, keeping track of the absolute error in the estimates. The code to do this was simulate(100, 0.30, 0.40, 1000). On average, the Lincoln index over- or under-estimated the number of bugs by about 16. This is a good estimate considering each tester greatly under-estimated the number of bugs.

If you didn’t think about using something like the Lincoln Index, in the previous example one tester would find around 30 bugs and the other around 40. The two lists might have 10 bugs in common, so you’d estimate the total number at 60, far short of 100. But the Lincoln index would often find estimates between 84 and 116.

Note that it is possible that the testers won’t find any of the same bugs. In that case the Lincoln Index cannot be computed and the code will divide by zero. But this is unlikely unless the p’s are small and n is small.

Software testing

Does the Lincoln Index actually provide a good bug count estimate? That depends on how well the assumptions are met. The index assumes all bugs are equally hard for a given tester to find. It does not assume that both testers are equally skilled, but it does assume that their chances of finding a bug are independent. In other words, tester A is no more or less likely to find a bug just because tester B found it.

The most questionable assumption is that all bugs are equally hard to find. That’s usually not true. But it may be true that all bugs of a certain kind are equally hard to find. For example, spelling errors may be easier to find than validation oversights, but the Lincoln Index might be good for estimating separately how many spelling errors or validation errors there are.

The index might provide a rough rule of thumb even if the assumptions it that go into it are violated. For example, suppose one tester found 15 bugs and another found 20. But only 3 of the bugs were the same. A naive estimate would say since there are 32 unique bugs found, there must be around that many in total. But the Lincoln Index would estimate 100 bugs. Maybe the Lincoln estimate is not at all accurate, but it does tell you to be worried that there may be a lot more bugs to find since the overlap between the two bug lists was so small.

