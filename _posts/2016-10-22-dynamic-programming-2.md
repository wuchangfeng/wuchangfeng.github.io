---
title: 用动态规划来解决问题-下
date: 2016-09-29 21:01:06
tags: 动态规划
---

本篇文章通过一个实例讲述如何利用动态规划解决单源最短路径的问题。

<!--more-->

如下所示路线图，我们的目的是要找出 A 到 E 之间最短的距离。

![](http://ww2.sinaimg.cn/large/b10d1ea5jw1f87ah5nhofj20j70arq3s.jpg)



我们仍然采用动态规划的思想来解决这个问题

**STEP 1**：描述最优解结构

用0到10分别表示这10个节点，要使得 0 到 10 之间的距离最短，令 f(i) 为到第 i 个节点的最短距离，则 **f(10) = min( f(7)+3 ,f(8)+4,f(9)+3 )**，同样的道理，我们得到 f(7),f(8),f(9).

**STEP 2** ：递归定义最优解的值

​			**f(i) = min(f(i)+Dij)**

其中 j 表示与 i 边有连接的点，并且 f(0) = 0;

**STEP3**：自底向上方式计算每个节点的最优值

最终我们利用递推公式分别求得 f(1) 到 f(10) 就好了。

按照动态规划的思想，我们在每一个阶段寻找新的节点的时候，一定要要求它是最优的子结构。

算法核心代码如下：

``` java
// 计算最短距离，并不包括计算其路径
public static int[] calMinDistance(int distance[][]) {  
        int dist[] = new int[distance.length];  
        dist[0] = 0;  
        for (int i = 1; i < distance.length; i++) {  
            int k = Integer.MAX_VALUE;  
            for (int j = 0; j < i; j++) {  
                if (distance[j][i] != 0) {  
                    if ((dist[j] + distance[j][i]) < k) {  
                        k = dist[j] + distance[j][i];  
                    }  
                }  
            }  
            dist[i] = k;  
        }  
        return dist;  
    }  
```

