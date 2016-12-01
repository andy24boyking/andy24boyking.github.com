---
layout: post
title: 微软2014实习生在线测试-题目3
category: code interview
tags: [microsoft]
---

# 题目3 : Reduce inversion count

时间限制:10000ms

单点时限:1000ms

内存限制:256MB

## Description

Find a pair in an integer array that swapping them would maximally decrease the inversion count of the array. If such a pair exists, return the new inversion count; otherwise returns the original inversion count.

Definition of Inversion: Let (A[0], A[1] ... A[n], n <= 50) be a sequence of n numbers. If i < j and A[i] > A[j], then the pair (i, j) is called inversion of A.

Example:<br/>
Count(Inversion({3, 1, 2})) = Count({3, 1}, {3, 2}) = 2<br/>
InversionCountOfSwap({3, 1, 2})=><br/>
{<br/>
 InversionCount({1, 3, 2}) = 1 <-- swapping 1 with 3, decreases inversion count by 1<br/>
 InversionCount({2, 1, 3}) = 1 <-- swapping 2 with 3, decreases inversion count by 1<br/>
 InversionCount({3, 2, 1}) = 3 <-- swapping 1 with 2 , increases inversion count by 1<br/>
}

<!-- excerpt -->

## Input

Input consists of multiple cases, one case per line.Each case consists of a sequence of integers separated by comma. 

## Output

For each case, print exactly one line with the new inversion count or the original inversion count if it cannot be reduced.


## 样例输入

    3,1,2
    1,2,3,4,5

## 样例输出

    1
    0

由于水平有限，测试的时候这道题我都没来得及做。这道题是给了我们一个整数数组，要求交换其中的一对数，使得产生的新数组中的逆序对数目最少，并输出新数组中逆序对的个数。

首先介绍一下**逆序对**的概念。在一个数组中，若序号靠前的一个数大于序号靠后的一个数，则称这两个数为一个逆序对。精确定义如下：设 A[1..n] 为一个包含 n 个整数的序列，若存在下标 i,j 满足 1 <= i < j<= n 且 A[i] > A[j] ，则 (A[i], A[j]) 被称作 A 的一个逆序对(题目中定义的是(i,j)为逆序对，这与常见定义不同，不过对于统计逆序对的数目没有影响)。

首先我们当然是要有一个求出给定数组中逆序对个数的算法。直观的思路自然是枚举数组中的每一个数，并将其与其后的每一个数来比较大小从而统计逆序对的数目，不过这样时间复杂度为 O(n<sup>2</sup>) ，显然不是个好方法。有一种借助归并排序的思想来处理的算法，它的思路是这样的：

假设一个数组可分为左右两部分，每部分内部均为非降有序的(如下图中的5、6、8 和 4、7)。我们将指针分别指向两部分的最后一个元素并比较它们的大小，8 大于 7 ，则说明 8 比右侧数组中的每个元素都大，因此可以和右侧中的每个元素都组成逆序对。右侧有 2 个元素，因此逆序对的个数加 2 ,同时将 8 写入辅助数组的末端并将左侧的指针向前移动一位。接下来比较当前左右部分的指针指向的元素 6 和 7 ，由于 6 小于 7 ,故此时不会增加逆序对数目，因此将 7 写入辅助数组并将右侧指针向前移动一位。重复上述过程直至辅助数组写满，我们可以发现这其实就是一个 2 路归并的过程，只是在归并的同时统计了逆序对的数目，此例中为 4 。因此我们可以开辟一个 O(n) 的辅助数组，借用归并排序的递归过程来统计一个普通整数数组中的逆序对个数。由于每一次的归并过程会将该子段变为非降有序的，从而消除了该段中的逆序对，因此递归的过程中不会出现逆序对数目的重复统计。该算法的时间复杂度即为 O(n*log*n) 。

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-04-13-merge.png)
一次归并的过程

现在有了性能较好的求逆序对数目的算法，再来看解决这道题还需要干什么。我们需要交换数组中的两个元素使得交换后的数组逆序对数目最小。我还没有找到什么高效的策略来选取待交换元素，因此只能够枚举了... 因此解决一行数据的时间复杂度即为 O(n<sup>3</sup>*log*n)。按照上述思路我实现的代码如下，没有经过OJ的测试，不知道能不能通过。

{% assign lang = 'c' %}
{% capture codeblock %}#include <stdio.h>
#include <string.h>

int merge(int copy[], int start, int end, int mid)
{
	int left[mid-start+1], right[end-mid];
	int i, n1 = mid-start, n2 = end-mid-1, n = end, count = 0;

	for (i = 0; i < mid-start+1; i++)
		left[i] = copy[i+start];
	for (i = 0; i < end-mid; i++)
		right[i] = copy[i+mid+1];

	while (n1 >= 0 && n2 >= 0) {
		if (left[n1] > right[n2]) {
			count += n2 + 1;
			copy[n--] = left[n1--];
		} else
			copy[n--] = right[n2--];
	}
	while (n1 >= 0)
		copy[n--] = left[n1--];
	while (n2 >= 0)
		copy[n--] = right[n2--];

	return count;
}

int merge_count(int copy[], int start, int end)
{
	if (start >= end)
		return 0;

	int mid = (start + end) / 2;
	int count_l = merge_count(copy, start, mid);
	int count_r = merge_count(copy, mid+1, end);
	int count = merge(copy, start, end, mid);

	return count_l + count_r + count;
}

int invertion_count(int A[], int length)
{
	int copy[length], i;

	for (i = 0; i < length; i++)
		copy[i] = A[i];

	return merge_count(copy, 0, length-1);
}

int main()
{
	char buff[1024];
	int A[50], i, j, len, count, temp;

	while (gets(buff) != NULL) {
		len = 0;
		char *p = strtok(buff, ",");
		while (p != NULL) {
			A[len++] = atoi(p);
			p = strtok(NULL, ",");
		}
		count = invertion_count(A, len);
		for (i = 0; i < len; i++) {
			for (j = i+1; j < len; j++) {
				temp = A[i];
				A[i] = A[j];
				A[j] = temp;
				temp = invertion_count(A, len);
				if (temp < count)
					count = temp;
				temp = A[i];
				A[i] = A[j];
				A[j] = temp;
			}
		}
		printf("%d\n", count);
	}

	return 0;
}
{% endcapture %}
{% include AH/print_code %}

输入数据中每一行的各个数字是用逗号分格，这里我用了 c 语言字符串库函数 strtok 来提取。不过这是个很奇怪的函数，很容易用错，最近我恰好用到了好几次，下次有空再详细讲讲。
