---
layout: post
title: hiho1：最长回文子串
category: code interview
tags: [string]
---

最长回文子串是一个很经典的问题，题面很简单。首先定义回文串，若一个字符串的正序和逆序完全相同，则称这个字符串为回文串，例如 "abaaba" 就是一个回文串。而对于一个给定的字符串，它的各个满足回文串定义的子串中最长的一个，即为它的最长回文子串。我们可以用该回文子串的起始位置和串长来描述它。因此，我们可以把问题描述如下：

给定一个字符串 *L* = *a*<sub>0</sub>*a*<sub>1</sub> ... *a*<sub>n-1</sub> ，若 *L* 的最长回文子串为 *L'* = *a*<sub>i</sub>*a*<sub>i+1</sub> ... *a*<sub>j</sub> (0 &le; i &le; j &le; n-1)。求出 *L'* 起始位置 *i* 以及 *L'* 的长度 *n* 。

<!-- excerpt -->

