---
title: AC自动机详解
date: 2018-07-01 20:09:55
categories: 算法
tags: 
- AC自动机
---
# 参考文献
[ac自动机最详细的讲解，让你一次学会ac自动机](https://blog.csdn.net/creatorx/article/details/71100840)

# 简介
AC自动机主要用来实现多模匹配。单模匹配类似于KMP算法，一个模式串对应一个主串；而多模匹配则是多个模式串对应多个主串，由于模式串个数不一定，主串长度及个数未知，如果使用KMP算法，则需要一个个去匹配，复杂度为O(k*m*(p+t)),k，m，p，t分别为模式串个数，主串个数，模式串长度，主串长度，复杂度非常高。而AC自动机通过建立字典树和使用失配指针，以及记录当前节点为后缀的模式串数，能够大大降低匹配次数。时间复杂度为建立字典树O(k*p),建立失配指针时间复杂度大概为O(k*p).匹配的时间复杂度为O(m*t).
## 数据结构

```
struct node{
    node *next[26];
    node *fail;
    int sum;
};
```
其中fail 是失配指针 
sum是这个节点是不是一个单词的结尾，以及相应的个数。 

# 算法步骤
1. 建立字典树


root节点为空节点。

```
void Insert(char *s)
{
    node *p = root;
    for(int i = 0; s[i]; i++)
    {
        int x = s[i] - 'a';
        if(p->next[x] == NULL)
        {
            newnode=(struct node *)malloc(sizeof(struct node));
            for(int j=0;j<26;j++) newnode->next[j] = 0;
            newnode->sum = 0;newnode->fail = 0;
            p->next[x]=newnode;
        }
        p = p->next[x];
    }
    p->sum++;
}
```

2. 构造fail指针

基于队列构造，q为队列，head为队首，tail为队尾。每次将每层节点入队。初始时，root进队列，root的子节点的失配指针指向root。

```
void build_fail_pointer()
{
    head = 0;
    tail = 1;
    q[head] = root;
    node *p;
    node *temp;
    while(head < tail)
    {
        temp = q[head++]; //当前结点
        for(int i = 0; i <= 25; i++)
        {
            if(temp->next[i])//对temp结点的下个结点建立失配指针
            {
                if(temp == root)
                {
                    temp->next[i]->fail = root; //初始化root的子结点
                }
                else
                {
                    p = temp->fail;
                    //如果有失配结点，则对temp的下一个结点有
                    //失配结点和无失配结点两种情况进行操作，无失配结点时，最终要到root处找，若果找不到
                    //则说明没有失配字符，将其指向root
                    while(p)
                    {
                        if(p->next[i])
                        {
                            temp->next[i]->fail = p->next[i];
                            break;
                        }
                        p = p->fail;
                    }
                    if(p == NULL) temp->next[i]->fail = root;
                }
                q[tail++] = temp->next[i];
            }
        }
    }
}
```

3. 利用fail指针进行匹配
 

对每个字符char，在字典树中查找，计算以字符char为结尾的总数目

```
void ac_automation(char *ch)
{
    node *p = root;
    int len = strlen(ch);
    for(int i = 0; i < len; i++)
    {
        int x = ch[i] - 'a';
        while(!p->next[x] && p != root) p = p->fail;
        p = p->next[x];
        if(!p) p = root;
        node *temp = p;
        while(temp != root)
        {
           if(temp->sum >= 0)
           {
               cnt += temp->sum; //cnt表示匹配总数
               temp->sum = -1;
           }
           else break;
           temp = temp->fail;
        }
    }
}
```
