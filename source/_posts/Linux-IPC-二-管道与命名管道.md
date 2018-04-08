---
title: Linux IPC(二)管道与命名管道
date: 2018-04-08 17:26:35
categories: Linux IPC
tags:
- Pipe 
- FIFO
---
## 管道 Pipe
从上一章,我们知道,管道通信一般用于具有亲缘关系的进程之间的通信.而且管道通信只支持单向通信,一方读的同时,另一方必须关闭读端操作的文件描述符,写操作同理.下面给出管道通信的一个例子.

```
#include <iostream>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#define MAXSIZE 1000
using namespace std;
int main(){
    int fd[2];
    if(pipe(fd) < 0){
        cout << "create pipe failed" << endl;
    }
    pid_t pid = fork();
    if(pid < 0){
        cout << "fork failed" << endl;
    }else if(pid == 0){
        //child process
        close(fd[0]);
        char buffer[MAXSIZE];
        memset(buffer, 0, sizeof(buffer));
        memcpy(buffer, "i am child\n", 11);
        for(int i = 0; i < 2; i++){
            write(fd[1],buffer,MAXSIZE);
            cout<<"child sent message:"<<buffer<<endl;
            sleep(2);
            }
    }else{
        //father process
        sleep(8);
        close(fd[1]);
        char recvbuffer[MAXSIZE];
        memset(recvbuffer, 0, sizeof(recvbuffer));
        for(int j = 0; j < 2; j++){
            read(fd[0],recvbuffer,sizeof(recvbuffer));
            cout<<"father recv message:"<<recvbuffer<<endl;
        }
        
    }
    
}
```
值得注意的是,管道的读写两端并不是任意的.一般fd[0]为读,fd[1]为写.
## 命名管道 FIFO
命名管道的好处在于不限于亲缘进程之间的通信.与Pipe共同之处是只支持单向通信.
可以使用以下两种方式创建命名管道.
```
#include <sys/types.h>  
#include <sys/stat.h>  
int mkfifo(const char *filename, mode_t mode);  
int mknod(const char *filename, mode_t mode | S_IFIFO, (dev_t)0);  
```
访问命名管道的常用方式

```
open(const char *path, O_RDONLY);//1  
open(const char *path, O_RDONLY | O_NONBLOCK);//2  
open(const char *path, O_WRONLY);//3  
open(const char *path, O_WRONLY | O_NONBLOCK);//4  
```
阻塞方式即在打开时必须等待命名管道文件的另一端有响应才会返回true,非阻塞方式即无需等待另一端即可返回.例如某个进程以只读方式打开fifo,阻塞方式下必须等待另一个进程以只写方式打开同一fifo才会返回true,非阻塞方式下直接返回true.

下面给出了一个命名管道的例子.
fifoWriter.cpp

```
/*本程序从一个文件中读取数据后写入命名管道*/
#include <iostream>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <limits.h>
using namespace std;
int main(){
    const char *fifo_name = "/home/luoshijie/lsj/myfifo";
    const char *file_name = "/home/luoshijie/lsj/myfile.txt";
    if(access(fifo_name,F_OK)<0){
        //fifo文件不存在,需要创建.
        if(mkfifo(fifo_name, 0755) < 0){
            cout << "create fifo failed" << endl;
        }
    }
    int fifo_fd, file_fd, n;
    fifo_fd = open(fifo_name, O_WRONLY);
    file_fd = open(file_name, O_RDONLY);
    if(fifo_fd < 0){
        cout << "open fifo_fd failed" << endl;
    }
    if(file_fd < 0){
        cout << "open file_fd failed" << endl;
    }
    char buffer[PIPE_BUF+1];
    memset(buffer, 0, sizeof(buffer));
    while((n=read(file_fd, buffer, PIPE_BUF))>0){
        write(fifo_fd, buffer, n);
    }
    close(fifo_fd);
    close(file_fd);
}
```
fifoReader.cpp

```
/*读取命名管道中的信息*/
#include <iostream>
#include <unistd.h>
#include <limits.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <fcntl.h>
using namespace std;
int main(){
    const char* fifo_name="/home/luoshijie/lsj/myfifo";
    int mkfifo_fd;
    mkfifo_fd = open(fifo_name, O_RDONLY);
    char buffer[PIPE_BUF+1];
    memset(buffer, 0, sizeof(buffer));
    while(read(mkfifo_fd, buffer, PIPE_BUF)){
        cout<<"buffer:"<<buffer<<endl;
    }
    close(mkfifo_fd);
}

```


