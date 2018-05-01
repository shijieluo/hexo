---
title: linux多线程
date: 2018-05-01 23:28:48
categories: Linux环境编程
tags:
- pthread
---
# Linux多线程
## 一个多线程的例子

```
//三个线程id分别为ABC,按ABC的顺序连续输出,重复10次
#include<iostream>
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
#include<unistd.h>
using namespace std;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
char tag = 'A';
void *thread_func(void *arg){
    char name = *((char *)arg);
    //cout << name << endl;     
    for(int i = 0; i< 10; i++){
        pthread_mutex_lock(&mutex);
        while(tag != name){ // 试试这时使用if会发生什么事
            cout << name << "wait" <<endl;
            pthread_cond_wait(&cond, &mutex);
        }        
        cout<< tag << i+1 <<endl;
        tag = (tag - 'A' + 1) % 3 + 'A';    
        pthread_mutex_unlock(&mutex);
        pthread_cond_broadcast(&cond); //试试用pthread_cond_signal会发生什么
        cout<< "awake"<< endl;        
    }
    pthread_exit(0);
    
}
int main(){
    pthread_t tid[3];
    char *temp = "ABC";
    for(int i = 0; i < 3; i++){        
        pthread_create(&tid[i], NULL, thread_func, (void *)(temp + i));
    }
    for(int i = 0; i< 3; i++){
        pthread_join(tid[i], NULL);
    }
    return 0;
}

```
## 对于pthread_cond_wait的解释
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex)
该函数需要配合互斥锁使用,进入等待之后将释放互斥锁,便于其它线程获取锁.

## 对于pthread_cond_signal 和 pthread_cond_broadcast的解释

这两个函数只有一个参数,pthead_cond_t.
它们的区别是,前者,只会激活先进入等待队列的第一个进程;后者会激活所有等待线程.

## while 和 if
这两个条件的区别
1. 因为pthread_cond_wait被激活后会返回,while会再执行一次循环体,这样如果条件依然没有被触发,将再次调用pthead_cond_wait;而if语句在被激活后会向下执行.
2. 上述不同可能导致死锁.