---
title: C++单例模式
date: 2018-07-01 20:18:33
categories: Design Pattern
tags:
---
# 单例模式
## 单线程下的单例

```
#include <iostream>
using namespace std;
class Singleton{
private:
	Singleton(){}
	static Singleton *p;
public:
	static Singleton *getInstance();
	
};
Singleton* Singleton::p = NULL;
Singleton* Singleton::getInstance(){
	if(p == NULL){
		p = new Singleton();
	}
	return p;
}	

```
以上是单线程下的单例模式，但是上述代码有很大的问题，考虑两个线程同时首次调用instance方法且同时检测到p是NULL值，则两个线程会同时构造一个实例给p，这是严重的错误！
## 多线程加锁实现（懒汉）
双加锁保证线程安全。

```
#include <iostream>
#include <pthread.h>
using namespace std;
class Singleton{
private:
	Singleton(){
		pthread_mutex_init(&mutex, NULL);
	}
	static Singleton *p;
	static pthread_mutex_t mutex;
public:
	static Singleton *getInstance();
	
};
Singleton* Singleton::p = NULL;
pthread_mutex_t Singleton::mutex;
Singleton* Singleton::getInstance(){
	if(p == NULL){
		pthread_mutex_lock(&mutex);
		if(p == NULL){
			p = new Singleton();
		}
		pthread_mutex_unlock(&mutex);
	}
	return p;
}	
```
## 多线程静态局部变量（懒汉）

```
#include <iostream>
#include <pthread.h>
using namespace std;
class Singleton{
private:
	Singleton(){
		pthread_mutex_init(&mutex,NULL);
	}
	static pthread_mutex_t mutex;
public:
	static Singleton* getInstance();
	
};
pthread_mutex_t Singleton::mutex;
Singleton* Singleton::getInstance(){
	pthread_mutex_lock(&mutex);
	static Singleton singleton;
	pthread_mutex_unlock(&mutex);
	return &singleton;
}
```
单例对象不存在时，两个线程同时想创建单例时，因为有锁机制的存在，所以不会初始化两次局部变量singleton。
## 多线程不加锁实现（饿汉）

```
#include <iostream>
using namespace std;
class Singleton{
private:
	Singleton(){}
	static Singleton* p;
public:
	static Singleton* getInstance();
	
};
Singleton* Singleton::p = new Singleton();
Singleton* Singleton::getInstance(){
	return p;
}
```
单线程下全局定义唯一一个单例对象。进入多线程环境后显然线程安全。


