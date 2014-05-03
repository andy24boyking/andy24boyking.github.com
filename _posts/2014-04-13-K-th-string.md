---
layout: post
title: 微软2014实习生在线测试-题目2
category: code interview
tags: [microsoft, string]
---
#题目2 : K-th string

时间限制:10000ms

单点时限:1000ms

内存限制:256MB

##Description

Consider a string set that each of them consists of {0, 1} only. All strings in the set have the same number of 0s and 1s. Write a program to find and output the K-th string according to the dictionary order. If such a string doesn’t exist, or the input is not valid, please output “Impossible”. For example, if we have two ‘0’s and two ‘1’s, we will have a set with 6 different strings, {0011, 0101, 0110, 1001, 1010, 1100}, and the 4th string is 1001.

<!-- excerpt -->

##Input

The first line of the input file contains a single integer t (1 ≤ t ≤ 10000), the number of test cases, followed by the input data for each test case.

Each test case is 3 integers separated by blank space: N, M(2 <= N + M <= 33 and N , M >= 0), K(1 <= K <= 1000000000). N stands for the number of ‘0’s, M stands for the number of ‘1’s, and K stands for the K-th of string in the set that needs to be printed as output.

##Output

For each case, print exactly one line. If the string exists, please print it, otherwise print “Impossible”.


##样例输入

    3
    2 2 2
    2 2 7
    4 7 47

##样例输出

    0101
    Impossible
    01010111011

这道题也不是很难，每行指定0的个数 N 和1的个数 M ，然后用 N 个 0 和 M 个 1 进行全排列，然后输出按字典序升序排列的第 K 个字符串，若 K 超过了全排列情况的总数目，则输出 “Impossible”。

由于题目所给 N 和 M 范围较小(2 <= N + M <= 33 且 N , M >= 0)，因此我直接使用比较熟悉的递归方式来生成全排列的各个情况。这道题是我唯一AC的一题。编写的代码如下：

{% assign lang = 'c' %}
{% capture codeblock %}#include <stdio.h>
#include <string.h>

unsigned int k;
unsigned int count = 0;
int flag;
char s[35];

void arrange(int n, int m, int i) 
{
    if (n > 0) {
        s[i] = '0';
        arrange(n-1, m, i+1);
    }
    if (m > 0) {
        s[i] = '1';
        arrange(n, m-1, i+1);
    }
    if(n + m == 0) {
        s[i] = '\0';
        ++count;
        if (count == k) {
            flag = 1;
            printf("%s\n", s);
        }
    } 
}

int main()
{
    int t, n, m, i;
        
    scanf("%d", &t);
    for (i = 0; i < t; i++) {
        scanf("%d %d %u", &n, &m, &k);
        flag = 0;
        count = 0;
        arrange(n, m, 0);
        if (flag == 0)
            printf("Impossible\n");
    }

    return 0;
}
{% endcapture %}
{% include AH/print_code %}

但是递归相对常用的算法如普通循环等，运行效率往往较低。而且在递归调用的过程当中系统为每一层的返回点、局部量等都开辟了栈来存储。递归次数过多容易造成栈溢出。对于全排列问题也可以采用非递归的算法全排列的非递归算法的思路是对于每一次的排列情况，通过变换某些元素的位置来得到升序的下一个排列的情况。

例如字符串 "c**b**dc**c**a" ，首先从后向前找到第一对相邻的递增字符对，在此例中，我们依次检查 "ca"、 "cc"、 "dc"、 "bd" ，则 "bd" 符合要求，其中 b 即为待替换的字符，它所在的位置称作**替换点**。接下来再从后往前，找到第一个比待替换的字符 b 大的字符 c ，即上面我标为粗体的两个字符，将它们交换。交换后的字符串为 "cc**dcba**"，接下来再将替换点之后的所有字符逆序，得到的字符串 "ccabcd" 即为 "cbdcca" 的下一个排列。特别的，当无法再从字符串中找到相邻递增字符对时，则说明当前字符串已为字典序的最大排列，全排列的所有情况已全部列出。采用非递归算法解决本题的代码如下：

{% assign lang = 'c' %}
{% capture codeblock %}#include <stdio.h>
#include <string.h>

void reverse(char str[], int start, int end)
{
    char temp;

    while (start < end) {
        temp = str[start];
        str[start] = str[end];
        str[end] = temp;
        start++;
        end--;
    }
}

int next_arrange(char str[], int len)
{
    int i = 0, pos = -1;
    char temp;

    for (i = len-1; i >= 1; i--)  {
        if (str[i-1] < str[i]) {
            pos = i - 1;
            break;
        }
    }
    if (pos < 0)
        return -1;
    for (i = len-1; i > pos; i--)  {
        if (str[i] > str[pos]) {
            temp = str[pos];
            str[pos] = str[i];
            str[i] = temp;
            reverse(str, pos+1, len-1);
            return 0;
        }
    }
    return -1;
}

void arrange(int n, int m, unsigned int k) 
{
    char str[35];
    int i = 0, len = n + m, count = 1;

    while (i < len) {
        if (n > 0) {
            str[i] = '0';
            n--;
        }else if (m > 0) {
            str[i] = '1';
            m--;
        }
        i++;
    }
    str[i] = 0;

    while (count < k) {
        if (next_arrange(str, len) < 0) {
            printf("Impossible\n");
            return;
        }
        count++;
    }
    printf("%s\n", str);
}

int main()
{
    int t, n, m, i;
    unsigned int k;
    char *str;

    freopen("2.input", "r", stdin);
        
    scanf("%d", &t);
    for (i = 0; i < t; i++) {
        scanf("%d %d %u", &n, &m, &k);
        arrange(n, m, k);
    }

    return 0;
}
{% endcapture %}
{% include AH/print_code %}