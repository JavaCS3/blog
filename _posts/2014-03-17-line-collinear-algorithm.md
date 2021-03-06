---
layout: post
category: algorithm
title: 平面内多点共线算法探究
---
## Perface

数据结构与算法可谓是计算机领域里面非常重要的知识。
最近我在[Coursera.org](http://www.coursera.org)温故温故这门课程。
今天就给大家分享一次作业题中的心得！

## Question

> Given a set of N distinct points in the plane, draw every (maximal) line segment that connects a subset of 4 or more of the points.
	
![pic](http://coursera.cs.princeton.edu/algs4/assignments/lines2.png)

## Solution

1. 暴力法

	思路如下：

    逐次选取平面上的四个点p, q, r, s判断这四个点是否共线。

    即:
   `p.slopeTo(q) == p.slopeTo(r) && p.slopeTo(r) == p.slopeTo(s)`

    (p.slopeTo(q) 为p, q 两点之间的斜率)

    分析：

    易知，此方法效率不高时间复杂度为`O(N^4)`而空间复杂度为`N`

    而且这种方法只能判断四点共线。当然这里还隐藏一个小问题：如果任意选取的四个点虽然共线，但是可能出现`q`在`p`左方`r`在`p`右方。那么线段起点和终点将不是`p -> s`，而可能是`q -> s`。对于这种情况，为了避免在选取好`p q r s`之后重复排序，可以将原始存放点的数组排序好（自然顺序）后再选取`p q r s`。至于，以什么形式排序，可以将点的`(x, y)`分割开来排序，例如：先按`y`方向比较大小，再按`x`方向比较大小，即可。

2. 先排序再判断
    
    思路如下：

    以`p`为原点，将其余的点按照斜率顺序排序。

    ![pic](http://coursera.cs.princeton.edu/algs4/assignments/lines1.png)

    如果有任意三个点（含以上）的与`p`点的斜率一致则将其打印出来。

    分析：

    瓶颈在于排序算法。基于比较的排序最快也是`N*log(N)`因此该算法的时间复杂度为`N^2*log(N)`空间复杂度为`N`

    然而，说得容易，做起来其实很困难！

## Details

1. 如何遍历每一个点？

    假设只有一个存储着点的数组`P[]`，那么每当新选中一个点`p`的时候当对其他的点进行按`p`斜率排序的时候`P[]`数组中的其它点的原始顺序就改变了。此时就很难保证，每次可以选中不同的点`p`。

    比如：

    排序前 

        P[] = { p0, p1, p2 }

    排序后 

        P[] = { p0, p2, p1 }

    解决方法：每一轮选中一个新的点`p`的时候，对原始`P[]`数组按自然顺序排序。这样，`p`点可以如此选择：`P[0], P[1], P[2] ...`，从而保证每次都会选中不同的`p`点。

2. 如何去除重复数据？
    
    实际编写过程中其实很容易会出现这种问题：

    假设原始点集数组为： 

        P[] = { p0, p1, p2, p3, p4, p5 }

    当以`p0`为原点按斜率排序后假设数组的顺序变为：

        P[] = { p0, p3, p4, p5, p1, p2 }

    对应的斜率为：

        P[] = { K0, k1, k1, k1, k2, k2 }

    可以发现，`p3 p4 p5`对于`p0`的斜率都为`K1`也就是`p0 p3 p4 p5`共线，假设点集的自然顺序为：

        p0 < p1 < p2 < p3 < p4 < p5

    那么输出的共线线段即为：

        p0 -> p3 -> p4 -> p5
    
    问题来了，如果选中的是`p3`呢？那么斜率排序后结果可能会是这样的

        p3 p0 p4 p5 p1 p2

    （这里稍微提一下，为什么每次选中的点都在排序后变为首个元素呢？这里可以利用一个小技巧，当可以令`p.slopeTo(p) = -Infinity`方可保证选中的元素在斜率排序后为首个元素）

    对于这种情况，输出的结果则会是：

        p3 -> p0 -> p4 -> p5

    这就是所谓的重复现象！对于此现象我在逛论坛的时候看到一些解决方法效率都不高。那么对于此方法应该如何应对呢？

    首先观察一下，正常情况下我们都希望输出是有序的，即：

        p0 -> p3 -> p4 -> p5

    可是乱序现象总是首元素不为最小，那么答案很自然了。

    一种简单可行的方法是：当要输出一个线段序列的时候，检查之后的顺序是否比首元素自然顺序小。如果存在那肯定是乱序，因此跳过这种情况即可！（能想到这一点确实不容易，我也是在论坛里看到别人的留言后顿悟的）

## Summery
    
使用先排序的方法确实会使效率提升不少！我的实现方法由于每次挑选新的`p`的过程中存在排序，因此时间复杂度为`O(N^2log(N) + Nlog(N)) = O(N^2log(N))`也算达到了预期吧！




