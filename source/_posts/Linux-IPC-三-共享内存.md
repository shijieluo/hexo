---
title: Linux IPC(三)共享内存
date: 2018-04-08 17:28:28
categories: Linux IPC
tags:
- 共享内存
---
# 共享内存通信
共享内存是linux本地通信的最快方式,即不同进程的不同用户空间的页指向同一个物理块.值得注意的是,共享内存没有提供同步机制,所以需要通过配合其它机制完成通信的同步.
## 共享内存的使用(system V共享内存)
1. shmget函数

```
#include <sys/ipc.h>
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);
```
第一个参数，与信号量的semget函数一样，程序需要提供一个参数key（非0整数），它有效地为共享内存段命名，shmget函数成功时返回一个与key相关的共享内存标识符（非负整数），用于后续的共享内存函数。调用失败返回-1.

不相关的进程可以通过该函数的返回值访问同一共享内存，它代表程序可能要使用的某个资源，程序对所有共享内存的访问都是间接的，程序先通过调用shmget函数并提供一个键，再由系统生成一个相应的共享内存标识符（shmget函数的返回值），只有shmget函数才直接使用信号量键，所有其他的信号量函数使用由semget函数返回的信号量标识符。

第二个参数，size以字节为单位指定需要共享的内存容量

第三个参数，shmflg是权限标志，它的作用与open函数的mode参数一样，如果要想在key标识的共享内存不存在时，创建它的话，可以与IPC_CREAT做或操作。共享内存的权限标志与文件的读写权限一样，举例来说，0644,它表示允许一个进程创建的共享内存被内存创建者所拥有的进程向共享内存读取和写入数据，同时其他用户创建的进程只能读取共享内存。
2. shmat函数和shmd函数

```
#include <sys/types.h>
#include <sys/shm.h>

void *shmat(int shmid, const void *shmaddr, int shmflg);

int shmdt(const void *shmaddr);

```
分别提供将进程的用户空间地址attach到共享内存和从共享内存detach的功能
3. shmctl函数

```
#include <sys/ipc.h>
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf);

struct shmid_ds {
               struct ipc_perm shm_perm;    /* Ownership and permissions */
               size_t          shm_segsz;   /* Size of segment (bytes) */
               time_t          shm_atime;   /* Last attach time */
               time_t          shm_dtime;   /* Last detach time */
               time_t          shm_ctime;   /* Last change time */
               pid_t           shm_cpid;    /* PID of creator */
               pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
               shmatt_t        shm_nattch;  /* No. of current attaches */
               ...
           };

```
第一个参数，shm_id是shmget函数返回的共享内存标识符。

第二个参数，command是要采取的操作，此操作与内核交互.它可以取下面的三个值:

- IPC_STAT：把shmid_ds结构中的数据设置为共享内存的当前关联值，即用共享内存的当前关联值覆盖shmid_ds的值。
- IPC_SET：如果进程有足够的权限，就把共享内存的当前关联值设置为shmid_ds结构中给出的值
- IPC_RMID：删除共享内存段

第三个参数，buf是一个结构指针，它指向共享内存模式和访问权限的结构。
shmid_ds结构至少包括以下成员：
struct shmid_ds记录了当前共享内存的各种信息. 
## 实例
创建一个页大小(4kB)的共享内存,实现shmWriter.cpp和shmReader.cpp的信息交互.</br>
shmWriter.cpp

```
/*system V共享内存*/
/*

       #include <sys/ipc.h>
       #include <sys/shm.h>

       int shmget(key_t key, size_t size, int shmflg);
       创建共享内存
       void *shmat(int shmid, const void *shmaddr, int shmflg);

       int shmdt(const void *shmaddr);
       
*/
#include <iostream>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
using namespace std;
int main(){
    void *shm = NULL;
    int shmid = shmget(1234, 4096, 0666|IPC_CREAT);
    if(shmid < 0){
        cout << "create shared memory failed..." << endl;
        exit(1);
    }
    shm = shmat(shmid, 0, 0);
    char s[] = "this is shmWriter..";
    strncpy(static_cast<char*>(shm), s, sizeof(s));
    if(shmdt(shm)){
        cout << "shm detaches failed..." << endl;
    }
    return 0;    
}
```
shmReader.cpp

```
/*shmReader.cpp*/
#include <iostream>
#include <unistd.h>
#include <stdlib.h>
#include <sys/shm.h>
#include <sys/ipc.h>
using namespace std;
int main(){
    void* shm = NULL;
    int shmid;
    shmid = shmget(1234, 4096, 0666 | IPC_CREAT);
    if(shmid < 0){
        cout << "create shared memory failed..." << endl;
        exit(1);
    }
    shm = shmat(shmid, 0, 0);
    cout << static_cast<char*>(shm) << endl;
    shmdt(shm);
    shmctl(shmid, IPC_RMID, 0);
    return 0;
}
```

## 共享内存的其它方式
1. Posix共享内存.使用shm_open()函数创建共享内存文件.
2. Posix内存映射文件,使用mmap()函数
