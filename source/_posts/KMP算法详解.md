---
title: KMP算法详解
date: 2018-07-01 20:14:24
categories: 算法
tags:
---
# 简介
KMP算法是一种改进的字符串匹配算法，由D.E.Knuth，J.H.Morris和V.R.Pratt同时发现，因此人们称它为克努特——莫里斯——普拉特操作（简称KMP算法）。KMP算法的关键是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的。具体实现就是实现一个next()函数，函数本身包含了模式串的局部匹配信息。时间复杂度O(m+n)。
KMP算法的流程是：
1. 主串的起始位置 = 上轮比配的起始位置 + 1
2. 模式串的其实位置 = 重复字符结构的下一位字符（若无重复子结构，则模式串的首字符）

## 什么是重复子结构
定义主串为T[0…n−1]，模式串为P[0…p−1]，则主串与模式串的长度各为n与p。
![image](https://images2015.cnblogs.com/blog/399159/201512/399159-20151231125119104-1124195921.png)
对于已匹配的字符，我们有：

```
T[i…i+j−1]=P[0…j−1]
```
KMP算法思想便是利用已经匹配上的字符信息，使得模式串的指针回退的字符位置能将主串与模式串已经匹配上的字符结构重新对齐。当有重复字符结构时，下一次匹配如下图所示：
![image](https://images2015.cnblogs.com/blog/399159/201512/399159-20151231125141510-1803698330.png)
从图中可以看出，下一次匹配开始时，主串指针在失配位置i+j，模式串指针回退到m+1；模式串的重复字符结构：

```
T[i+j−m−1…i+j−1]=P[j−m−1…j−1]=P[0…m]
T[i+j]≠P[j]≠P[m+1]
```
所以，重复子结构即模式串的前缀和后缀一致的结构。

## 部分匹配函数
根据上面的讨论，我们定义部分匹配函数（Partial Match，在数据结构书[2]称之为失配函数）：


```math
f(j)=\begin{cases}
max\lbrace m \rbrace,
&\text{P[j-m-i...j-1]=P[0...m]}  
\\ -1, & \text{else} 
\end{cases}
```
其表示字符串P[0…j]的前缀与后缀完全匹配的最大长度，也表示了模式串中重复字符结构信息。KMP中大名鼎鼎的next[j]函数表示对于模式串失配位置j+1，下一轮匹配时模式串的起始位置（即对齐于主串的失配位置）；则

```math
next[j]=f(j)+1
```
为什么要取最大的m值？

因为m值如果取的过小，将导致模式串移动的位置过大，因而忽略了本来能够匹配的串。
# 代码实现
求next数组的C++实现代码：

```
vector<int> getNext(string p){
    vector<int> next(p.size());
    next[0] = -1;  
    int k = -1;  
    int j = 0;  
    while (j < p.size()-1)  
    {  
        //p[k]表示前缀，p[j]表示后缀  
    if (k == -1 || p[j] == p[k])   
        {  
            ++k;  
            ++j;  
            next[j] = k;  
        }  
        else   
        {  
            k = next[k];  
        }  
    }
    return next;
}

```
为何递归前缀索引k = next[k]，就能找到长度更短的相同前缀后缀呢？这又归根到next数组的含义。我们拿前缀 p0 pk-1 pk 去跟后缀pj-k pj-1 pj匹配，如果pk 跟pj 失配，下一步就是用p[next[k]] 去跟pj 继续匹配，如果p[ next[k] ]跟pj还是不匹配，则需要寻找长度更短的相同前缀后缀，即下一步用p[ next[ next[k] ] ]去跟pj匹配。此过程相当于模式串的自我匹配，所以不断的递归k = next[k]，直到要么找到长度更短的相同前缀后缀，要么没有长度更短的相同前缀后缀。

KMP的C++实现代码：

```
int kmp(string s, string p, vector<int> next)  
{  
    int i = 0;  
    int j = 0;  
    int sLen = s.size();  
    int pLen = p.size();  
    while (i < sLen && j < pLen)  
    {  
        //①如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++      
        if (j == -1 || s[i] == p[j])  
        {  
            i++;  
            j++;  
        }  
        else  
        {  
            //②如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]      
            //next[j]即为j所对应的next值        
            j = next[j];  
        }  
    }  
    if (j == pLen)  
        return i - j;  
    else  
        return -1;  
}  
```
# 参考文献
* [【模式匹配】KMP算法的来龙去脉](http://www.cnblogs.com/en-heng/p/5091365.html)
* [从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827#)




