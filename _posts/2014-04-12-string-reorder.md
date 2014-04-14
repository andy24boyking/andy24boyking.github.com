---
layout: post
title: 微软2014实习生在线测试-题目1
category: code interview
tags: [microsoft, string]
---
今天参加了微软实习生招聘的“笔试”，即在线测试环节，英文描述的4道编程题，比赛时间为2个小时。形式为在线提交代码（支持C、C++、Java、C#四种语言），由系统进行测试来评判你的代码是否通过(AC)。由于我比较水，只通过其中的第二题。第一题的题目如下：

#题目1 : String reorder

时间限制:10000ms

单点时限:1000ms

内存限制:256MB

##Description

For this question, your program is required to process an input string containing only ASCII characters between ‘0’ and ‘9’, or between ‘a’ and ‘z’ (including ‘0’, ‘9’, ‘a’, ‘z’).

Your program should reorder and split all input string characters into multiple segments, and output all segments as one concatenated string. The following requirements should also be met,
1. Characters in each segment should be in strictly increasing order. For ordering, ‘9’ is larger than ‘0’, ‘a’ is larger than ‘9’, and ‘z’ is larger than ‘a’ (basically following ASCII character order).
2. Characters in the second segment must be the same as or a subset of the first segment; and every following segment must be the same as or a subset of its previous segment.

Your program should output string “<invalid input string>” when the input contains any invalid characters (i.e., outside the '0'-'9' and 'a'-'z' range).

<!-- excerpt -->

##Input

Input consists of multiple cases, one case per line. Each case is one string consisting of ASCII characters.

##Output

For each case, print exactly one line with the reordered string based on the criteria above.


##样例输入

    aabbccdd
    007799aabbccddeeff113355zz
    1234.89898
    abcdefabcdefabcdefaaaaaaaaaaaaaabbbbbbbddddddee

##样例输出

    abcdabcd
    013579abcdefz013579abcdefz
    <invalid input string>
    abcdefabcdefabcdefabdeabdeabdabdabdabdabaaaaaaa

第一题看起来比较简单，输入为每行一个字符串，当字符串包含有数字_'0'~'1'_和小写字母_'a'~'z'_以外的字符时则视为非法，输出一行 `<invalid input string>` 。否则，按照 ascII 码里字符递增的顺序，将输入的字符串重组输出。相当于将字符串分为了若干段，其中每一段内部都是按照字符ascII码递增的顺序排列，相邻的段之间，靠后的段必为前一段的子串（包括两段相同的情形）。可参照上方所给的输入输出样例来直观体会。

我的思路是定义一个256长度的整型数组 `freq[256]` 来记录每一个ascII码所在字符串中出现的频率，在统计频率的过程如遇到非法字符，则直接判错输出错误信息。虽然题目中所给合法字符只有36种，定义256长度的数组有点浪费，不过这样的话 freq 数组的下标对应的即为ascII码值，写代码较为简单。对于合法的字符串，输出时即一趟趟遍历 freq 数组中数字_'0'~'1'_和小写字母_'a'~'z'_的部分，输出计数值大于0的下标（即对应字符），同时将该下标计数值减一。直到某一趟遍历时 freq 数组已全为0时结束。编写的代码如下：

{% assign lang = 'c' %}
{% capture codeblock %}#include <stdio.h>
#include <string.h>

#define MAX 4096
char str[MAX];

int main()
{
    int i, n, flag = 0, flag2 = 0;
    int freq[256] = {0};

    while (scanf("%s", str) != EOF) {
        n = strlen(str);
        flag = 0;
        for (i = 0; i < 256; i++)
            freq[i] = 0;
        for (i = 0; i < n; i++) {
            freq[str[i]]++;
            if ((str[i] < '0' || str[i] > '9') && (str[i] < 'a' || str[i] > 'z')) {
                flag = 1;
                break;
            }
        }
        if (flag == 0) {
            while (1) {
                flag2 = 0;
                for (i = 0; i < 256; i++) {
                    if (freq[i] != 0) {
                        flag2 = 1;
                    } 
                }
                if (flag2 == 1) {
                    for (i = '0'; i <= '9'; i++) {
                        if (freq[i] > 0) {
                            printf("%c", i);
                            freq[i]--;
                        }
                    }
                    for (i = 'a'; i <= 'z'; i++) {
                        if (freq[i] > 0) {
                            printf("%c", i);
                            freq[i]--;
                        }
                    }
                } else {
                    printf("\n");
                    break;
                }
            }
        } else {
            printf("<invalid input string>\n");
        }
        memset(str, 0, MAX);
    }

    return 0;
}
{% endcapture %}
{% include AH/print_code %}

自认为代码没什么问题，自测了几组数据也都正确。不过测评结果总是“90/100 WA”，应该是有一组数据一直没有通过。因为题目中没有给每行字符串的最大数目，我最开始以为是这个问题，可是我将代码中的MAX值一直加到200M(题目限制内存256M)都还是得到同样的测试结果，我就实在没办法了。希望AC了这道题的朋友能够给我一些建议。