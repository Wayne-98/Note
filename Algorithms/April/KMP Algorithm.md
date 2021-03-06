---
title: KMP Algorithm
date: {date}
tags: Algorithms
categories: Algorithms
---
字符串的前缀：除最后一个字符以外，字符串的所有头部子串。

字符串的后缀：除第一个字符以外，字符串的所有尾部子串。

部分匹配值：字符串的前缀和后缀的最长相等前后缀长度。

* ‘a’ 的前缀和后缀都为空集，最长相等前后缀长度为0
* ‘aba’ 的前缀为 {a, ab}, 后缀为{a, ba}, 最长相等前后缀长度为1



移动位数 = 已匹配的字符数 - 对应的部分匹配值

如下图所示，当 c 与 b 不匹配时，已匹配 ‘abca' 的前缀 a 和后缀 a 为最长的公共元素。已知前缀 a 与 b、c 均不同，与后缀 a 相同，故无须比较，直接将子串移动“已匹配的字符数 - 对应的部分匹配值”，用子串前缀后面的元素与主串匹配失败的元素开始比较即可。

![](https://github.com/Wayne-98/image/blob/master/Algorithms/KMP%20next.png?raw=true)

next数组的求解方法

