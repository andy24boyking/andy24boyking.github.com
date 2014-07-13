---
layout: post
title: 最大和路径
category: algorithm
tags: [dynamic programming, project euler]
---
[**欧拉计划** ](https://projecteuler.net/) (Project Euler) 是一个解题网站，网站上提供了一系列可以用编程方法来解决的各种各样的数学问题，因此该网站的受众主要以对数学和计算机编程感兴趣的人。每道题的描述最终都会归结为提交一个数字作为答案，系统 check 你提交的数字来判断是否完成此题，其他的一概不管。不过网站上也给出了说明，原则上来说每道题的解题程序都应当能在一分钟之内算得结果。原本网站还提供了登录帐号，可以记录所有已经解答出的题目题号，有一定积累之后看着相当有成就感的。而且还为每道题提供了一个简单的论坛供全世界各地的人进行交流。可惜由于前段时间网站被黑，这些功能都暂时关闭了，[点这里](https://projecteuler.net/news)了解详情。

闲扯了这么多，言归正传。来看里面的 [Problem 18](https://projecteuler.net/problem=18) 和 [Problem 67](https://projecteuler.net/problem=67)。
两道题其实是一道题，只是数据规模不同，求解最大和。如下所示，从三角形的顶端出发，可以向左下或者右下移动，直至三角形底部为止，计算使得途经各数之和的最大值。下图例子中即为加粗数字之和，3 + 7 + 4 + 9 = 22 。

<!-- excerpt -->

{:.post-image}
**3**<br/>
**7** 4<br/>
2 **4** 6<br/>
8 5 **9** 3

第 18 题所给的三角形如下:

{:.post-image}
75<br/>
95 64<br/>
17 47 82<br/>
18 35 87 10<br/>
20 04 82 47 65<br/>
19 01 23 75 03 34<br/>
88 02 77 73 07 63 67<br/>
99 65 04 28 06 16 70 92<br/>
41 41 26 56 83 40 80 70 33<br/>
41 48 72 33 47 32 37 16 94 29<br/>
53 71 44 65 25 43 91 52 97 51 14<br/>
70 11 33 28 77 73 17 78 39 68 17 57<br/>
91 71 52 38 17 14 91 43 58 50 27 29 48<br/>
63 66 04 68 89 53 67 30 73 16 69 87 40 31<br/>
04 62 98 27 23 09 70 98 73 93 38 53 60 04 23

如果说用暴力破解的方法，计算每一种可能的路径的和，那么一共会有 *n*! 种情况，n 为三角形的行数。这样做效率会很低。我们先来分析一下这个问题，任取第 i 行第 j 列的数 A[*i*, *j*]，其中 1 &le; j &le; i &le; n 。假如一条路径经过 A[*i*, *j*]，那么它的上一结点只会是 A[*i-1*, *j*] 或者 A[*i-1*, *j-1*] (1 < *j* < *i*)；特别的，如果 A[*i*, *j*] 位于三角形的两边，则只有一种选择，即 A[*i*, 1] 上一结点是 A[*i-1*, 1]， A[*i*, *i*] 上一结点是 A[*i-1*, *i-1*] 。设 m[*i*, *j*] 为从三角形顶部到结点 A[*i*, *j*] 处的最大的途径结点之和。初始值 m[1, 1] = A[1, 1] ；对非顶点的任意 A[*i*, *j*]， 由以上分析可知路径只会来源于上方两个点之一，因此有 m[*i*, *j*] = *max* { m[*i* - 1, *j* - 1], m[*i* - 1, *j*]} + A[*i*, *j*] ，同样当 A[*i*, *j*] 位于三角形两侧边上时，则只有一种选择。将上述分析整理为如下递归公式：

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-07-03-formula1.png)

因此整个三角形的最大路径和则为最后一行的各个 m 值的最大值：

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-07-03-formula2.png)

直接用该递归公式求解，会得到暴力破解的情形，每一条路径的和都会被计算一次。但实际上有很多结点 m 值被重复计算，我们可以保存下每一个结点的 m 值来避免重复计算。

假设三角形数据存放在二维数组 `unsigned int data[N][N]` 中，二维数组 `unsigned int sum[N][N]` 用来存放每一个结点的当前最大和，`int path[N][N]` 则用来存放当前结点的最大和路径来自上一层的第几个结点，以便于构造最优解的路径 (实际上以上三个数组都只用了一半的存储空间)。求取每个结点的当前最大和的算法实现如下：

{% assign lang = 'c' %}
{% capture codeblock %}void max_path_sum(unsigned int data[N][N], unsigned int sum[N][N],
    int path[N][N])
{
    int i, j, k;

    sum[0][0] = data[0][0];
    for (i = 1; i < N; i++) {
        sum[i][0] = sum[i-1][0] + data[i][0];
        path[i][0] = 0;
        for (j = 1; j < i; j++) {
            k = sum[i-1][j-1] > sum[i-1][j] ? j-1 : j;
            sum[i][j] = sum[i-1][k] + data[i][j];
            path[i][j] = k;
        }
        sum[i][i] = sum[i-1][i-1] + data[i][i];
        path[i][i] = i-1;
    }
}
{% endcapture %}
{% include AH/print_code %}

利用 sum 数组存放子问题中各个结点的当前最大和，每个结点只会计算一次，从而避免了递归算法中重复计算的问题。该算法的时间复杂度为 O(*n*<sup>2</sup>) 。

如果要求解这道问题，执行完 max_path_sum 后遍历一遍 sum 数组的最后一行输出最大值即可。若要输出最大和的完整路径，则要用到记录的 path 数组，实现如下：

{% assign lang = 'c' %}
{% capture codeblock %}void print_path(unsigned int *max_sum, unsigned int sum[N][N], int path[N][N])
{
    *max_sum = 0;
    int i, k;

    for (i = 0; i < N; i++) {
        if (sum[N-1][i] > *max_sum) {
            *max_sum = sum[N-1][i];
            k = i;
        }
    }
    __print_path(path, k, N-1);
    printf("\n");
}

void __print_path(int path[N][N], int k, int i)
{
    if (i == 0) {
        printf("%d", k);
        return;
    } else {
        __print_path(path, path[i][k], i-1);
        printf("->%d", k);
    }
}
{% endcapture %}
{% include AH/print_code %}

可解得[问题 18](https://projecteuler.net/problem=18) 的最大和为 1074，打印出来的路径如下(打印的为二维数组的列下标)：

    0->1->2->2->2->3->3->3->4->5->6->7->8->8->9

最近在看动态规划算法，这道题就是一个典型的例子，它具有最优子结构，也满足子问题重叠的条件，因此可以分析出问题求解的递归关系，然后通过保存子问题的解来减小问题求解的时间复杂度。