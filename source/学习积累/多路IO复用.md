---
title: 多路IO复用
date: 2022-03-01 11:06:58
type: 基础知识
categories:
- 开源项目
  tags:
  thumbnail: https://tva1.sinaimg.cn/large/008i3skNgy1gt14d2z4q0j31900u0tb5.jpg
---
# 阻塞I/O

阻塞，就是调用我（函数），我（函数）没有接收完数据或者没有得到结果之前，我不会返回。

在linux中，默认情况下所有的socket都是阻塞的，一个典型的读操作流程大概是这样：

![img](https://pics6.baidu.com/feed/6159252dd42a283467eca28dec9161ed14cebf9e.jpeg?token=b2a72b2fc54ef69803195f323f2c08ae)



当用户进程调用了**read()/recvfrom()** 等系统调用函数，它会进入内核空间中，当这个网络I/O没有数据的时候，内核就要等待数据的到来，而在用户进程这边，整个进程会被阻塞，直到内核空间返回数据。

当内核空间的数据准备好了，它就会将数据从内核空间中拷贝到用户空间，此时用户进程才解除阻塞的的状态，重新运行起来。

**这里的阻塞指的是空转**，一直占用cpu，while(true)

所以，阻塞I/O的特点就是在IO执行的两个阶段（用户空间与内核空间）都被阻塞了。

# 非阻塞I/O

非阻塞，就是调用我（函数），我（函数）立即返回。阻塞调用是指调用结果返回之前，**当前线程会被挂起**，释放cpu（线程进入非可执行状态，在这个状态下，cpu不会给线程分配时间片，即线程暂停运行）。函数只有在得到结果之后才会返回。

**有人也许会把阻塞调用和同步调用等同起来**，

![img](https://pics1.baidu.com/feed/18d8bc3eb13533fa8568e10c11f7551840345be5.jpeg?token=5d94d2a825d48c613a7f569baa5cf678)

进程会反复调用recvfrom,此为系统调用。当返回成功时，唤醒执行线程，处理数据。

**能看到，非阻塞I/O的特点是用户进程需要不断的 主动询问 内核空间的数据准备好了没有。**

在操作系统中，程序运行的空间分为内核空间和用户空间，用户空间所有对io操作的代码（如文件的读写、socket的收发等）都会通过系统调用进入内核空间完成实际的操作。

## 同步I/O

而且我们都知道CPU的速度远远快于硬盘、网络等I/O。在一个线程中，CPU执行代码的速度极快，然而，一旦遇到I/O操作，如读写文件、发送网络数据时，就需要等待 I/O 操作完成，才能继续进行下一步操作，这种情况称为**同步 I/O**。

**其实所谓同步，就是在发出一个功能调用时，在没有得到结果之前，该调用就不返回。也就是必须一件一件事做,等前一件做完了才能做下一件事。**

实际工作中我们却很少使用同步I/O，因为当你读写某个文件，进行I/O操作时候，如果数据没有及时回应到，那么系统就会将当前执行读写的线程挂起来等待数据的读取完成，而其他需要CPU执行的代码就无法被当前线程执行，这就是同步I/O的弊端。

仅仅因为一个I/O操作就会阻塞当前线程，导致其他代码无法执行，当然我们遇到这样时候会选择用多线程或者多进程来并发执行代码。

但是多线程和多进程也无法根除这种阻塞问题，因为系统内存大小的限制，所以系统不能无限的增加线程和进程。此外过多的线程和进程，就会导致系统切换线程和进程的开销变大，真正运行代码时间就会变少，这样子系统性能也会严重下降。

## 异步I/O

简单来说就是，用户不需要等待内核完成实际对io的读写操作就直接返回了。

当一个异步过程调用发出后，调用者不能立刻得到结果。**实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者**。

**I/O过程主要分两个阶段：**

1.数据准备阶段

2.内核空间复制回用户进程缓冲区空间

无论**阻塞式IO**还是**非阻塞式IO**，都是**同步IO模型**，

返回是啥意思？应该是挂起

区别就在与**第一步是否完成后才返回**，但第二步都需要当前进程去完成，

异步IO呢，就是**从第一步开始就返回**，**直到第二步完成后才会返回一个消息**，也就是说，异步能够让你在第一步时去做其它的事情。

![img](https://pics0.baidu.com/feed/b3b7d0a20cf431ad65a31580f21204a82fdd98c6.jpeg?token=c8d2723fde7fc56005b81484e843f305)

从图中可以看出，异步是一种被动接受消息的过程

而同步是一种主动的形式，阻塞IO是一直询问，非阻塞IO是隔一段时间就询问

同步IO和异步IO的区别就在于：**数据拷贝的时候进程是否阻塞**

阻塞IO和非阻塞IO的区别就在于：**应用程序的调用是否立即返回**

即立即退出本调用。

因为异步IO把IO的操作给了**内核，让内核去操作**，内核操作完了通知一下

同步IO的话，需要等待IO操作从内核态的数据缓冲区拷贝到用户态的数据缓冲区，所以此时的同步IO是阻塞的。

## 多路复用I/O



# 多路IO复用

多路复用I/O就是我们说的 **select，poll，epoll** 等操作，复用的好处就在于 单个进程 就可以同时处理 多个 网络连接的I/O，

能实现这种功能的原理就是 select、poll、epoll 等函数会不断的 轮询 它们所负责的所有 socket ，当某个 socket 有数据到达了，就通知用户进程。

一般在Linux下我们会有以下几种的字符设备读写方式，下面是一个使用的对比：

**1、查询方法：一直在查询，不断去查询是否有事件发生，整个过程都是占用CPU资源，非常消耗CPU资源。**

**2、中断方式：当有事件发生时，就去跳转到相应事件去处理，CPU占用时间少。**

**3、poll方式: 中断方式虽然占用CPU资源少，但是在应用程序上需要不断在死循环里面执行读取函数，应用程序不能去做其它事情。poll机制解决了这个问题，当有事件发生时，才去执行读read函数，按键事件没有按下时<如果规定了时间，超过时间后返回无按键信息>，去执行其它的处理函数。**

I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。

**但select，poll，epoll本质上都是同步I/O**，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个**读写过程是阻塞的**，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

单线程使用select()/poll()等系统调用实现对多个文件句柄的监视，如果有某个文件句柄的事件发生就会返回，

没有的话就会阻塞等待，单其实select（）和poll（）都是表现为外部阻塞式，内部非阻塞式自动轮询多路阻塞式IO。

我们再说一下**select,poll和epoll这几个IO复用方式**，这时你就会了解它们为什么是同步IO了，以epoll为例，以epoll为例，在epoll开发的服务器模型中，**epoll_wait**()这个函数会阻塞等待就绪的fd，将就绪的fd拷贝到epoll_events集合这个过程中也不能做其它事。（虽然这段时间很短，所以epoll配合非阻塞IO是很高效也是很普遍的服务器开发模式--同步非阻塞IO模型）。

有人把epoll这种方式叫做同步非阻塞（NIO），因为用户线程需要不停地轮询，自己读取数据，看上去好像只有一个线程在做事情，也有人把这种方式叫做异步非阻塞（AIO）

**因为毕竟是内核线程负责扫描fd列表，并填充事件链表的，**

个人认为真正理想的异步非阻塞，应该是内核线程填充事件链表后，主动通知用户线程，

或者调用应用程序事先注册的**回调函数**来处理数据，

如果还需要用户线程**不停的轮询**来获取事件信息，就不是太完美了，所以也有不少人认为epoll是伪AIO，还是有道理的。



## select（）函数使用示例

单个线程打开的FD是有限的，通常默认是1024个，可以通过FD_SETSIZE设置

每次调用select（）都需要把fd集合从用户态拷贝到内核态，在需要轮询的fd很多时，会造成较大的性能开销。

![img](https://pics1.baidu.com/feed/c995d143ad4bd113139bacbced8b0c084afb0591.jpeg?token=8d78992c7ca08b2775982298df798b5e)

select（）前面提到了select（）是通过轮询扫描检测事件有没有发生的所以执行效率比较低。

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/time.h>
int main(void)
{
    int fd=-2;
    char buf[20];
    fd_set set;
    struct timeval tm;
    bzero(buf,sizeof(buf));
    fd=open("/dev/input/mouse0",O_RDONLY);
    if (fd<0)
    {
        perror("open:");
    }
    printf("already read\n");
    tm.tv_sec=10;   //设置等待超时时间
    tm.tv_usec=0;
    FD_ZERO(&set);   //清空set变量
    FD_SET(fd,&set); //将需要监视的fd加入到select（）的见识列表
    FD_SET(0,&set);
    while (1)
    {
        int ret=select(fd+1,&set, NULL,NULL, &tm);
        if (ret<0)
        {
            perror("why");
        }
        else if (ret==0)
        {
            printf("等待超时\n");
        }
        else
        {
            if(FD_ISSET(fd,&set)) //判断时哪一个被监视的文件句柄事件发生了
            {
                read(fd,buf,5);
                printf("read mouse0 buf=%s\n",buf);
            }
            if(FD_ISSET(0,&set))
            {
                read(0,buf,5);
                printf("read keyborad buf=%s\n",buf);
            }
        }
    }
    return 0;
}
```

## poll（）函数使用示例

poll（）函数与select（）函数一样每次调用都需要把fd集合从用户态拷贝到内核态，在需要轮询的fd很多时，会造成较大的性能开销。

poll（）对文件句柄的监视也是和select（）函数一样采用线性扫描的方式进行轮询，所以在高并发的情况下效率也是比较底下的。

但是poll（）与select（）不同的一点就是poll没有监视数量的限制.(select是1024个)

**select的几大缺点：**

**（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大**

**（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大**

**（3）select支持的文件描述符数量太小了，默认是1024**

**poll**的机制与**select**类似，与select在本质上没有多大差别，管理多个描述符也是进行轮询，

**根据描述符的状态进行处理，但是poll没有最大文件描述符数量的限制。**

**poll和select同样存在一个缺点就是**，包含**大量文件描述符的数组被整体复制于用户态和内核的地址空间之间**，而**不论这些文件描述符是否就绪**，它的**开销随着文件描述符数量的增加而线性增大。**

**epoll**是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。

**epoll使用一个文件描述符管理多个描述符**，将用户关系的文件描述符的事件存放到**内核的一个事件表**中，

这样在用户空间和内核空间的copy只需一次。

### 事件

首先介绍下如何使用事件。事件Event实际上是个内核对象，它的使用非常方便。

下面列出一些常用的函数。

第一个 CreateEvent创建事件,函数原型：

```c

HANDLECreateEvent(

 LPSECURITY_ATTRIBUTES lpEventAttributes,

 BOOL bManualReset,

 BOOL bInitialState,

 LPCTSTR lpName

);
```

函数说明：

第一个参数表示安全控制，一般直接传入NULL。

第二个参数确定事件是手动置位还是自动置位，传入TRUE表示手动置位，传入FALSE表示自动置位。

如果为自动置位，则对该事件调用WaitForSingleObject()后会自动调用ResetEvent()使事件变成未触发状态。

打个小小比方，**手动置位事件**相当于教室门，教室门一旦打开（被触发），所以有人都可以进入直到老师去关上教室门（事件变成未触发）。

**自动置位事件**就相当于医院里拍X光的房间门，门打开后只能进入一个人，这个人进去后会将门关上，其它人不能进入除非门重新被打开（事件重新被触发）。

第三个参数表示事件的初始状态，传入TRUR表示已触发。

第四个参数表示事件的名称，传入NULL表示匿名事件。

第二个 OpenEvent

函数功能：根据名称获得一个事件句柄。

函数原型：

```c
HANDLEOpenEvent(

 DWORD dwDesiredAccess,

 BOOL bInheritHandle,

 LPCTSTR lpName   //名称

);
```

第三个SetEvent

函数功能：触发事件

函数原型：BOOLSetEvent(HANDLEhEvent);

函数说明：每次触发后，必有一个或多个处于等待状态下的线程变成可调度状态。

即释放资源，使得某些线程达到就绪状态

 第四个ResetEvent

函数功能：将事件设为末触发

函数原型：BOOLResetEvent(HANDLEhEvent);

最后一个事件的清理与销毁

由于事件是内核对象，因此使用CloseHandle()就可以完成清理与销毁了。

在经典多线程问题中**设置一个事件和一个关键段**。用事件处理主线程与子线程的同步，用关键段来处理各子线程间的互斥。详见代码：

```c
#include <stdio.h>
#include <process.h>
#include <windows.h>
long g_nNum;
unsigned int __stdcall Fun(void *pPM);
const int THREAD_NUM = 10;
//事件与关键段
HANDLE  g_hThreadEvent;
CRITICAL_SECTION g_csThreadCode;
int main()
{
	printf("     经典线程同步 事件Event\n");
	printf(" -- by MoreWindows( http://blog.csdn.net/MoreWindows ) --\n\n");
	//初始化事件和关键段 自动置位,初始无触发的匿名事件
	g_hThreadEvent = CreateEvent(NULL, FALSE, FALSE, NULL); 
	InitializeCriticalSection(&g_csThreadCode);
 
	HANDLE  handle[THREAD_NUM];	
	g_nNum = 0;
	int i = 0;
	while (i < THREAD_NUM) 
	{
		handle[i] = (HANDLE)_beginthreadex(NULL, 0, Fun, &i, 0, NULL);
		WaitForSingleObject(g_hThreadEvent, INFINITE); //等待事件被触发
		i++;
	}
	WaitForMultipleObjects(THREAD_NUM, handle, TRUE, INFINITE);
 
	//销毁事件和关键段
	CloseHandle(g_hThreadEvent);
	DeleteCriticalSection(&g_csThreadCode);
	return 0;
}
unsigned int __stdcall Fun(void *pPM)
{
	int nThreadNum = *(int *)pPM; 
	SetEvent(g_hThreadEvent); //触发事件
	
	Sleep(50);//some work should to do
	
	EnterCriticalSection(&g_csThreadCode);
	g_nNum++;
	Sleep(0);//some work should to do
	printf("线程编号为%d  全局资源值为%d\n", nThreadNum, g_nNum); 
	LeaveCriticalSection(&g_csThreadCode);
	return 0;

```



![image-20220506020754360](source/学习积累/多路IO复用.assets/image-20220506020754360.png)

可以看出来，经典线线程同步问题已经圆满的解决了——线程编号的输出没有重复，说明主线程与子线程达到了同步。全局资源的输出是递增的，说明各子线程已经互斥的访问和输出该全局资源。



### 代码演示

在poll的使用过程中当程序开始运行等待超时后，如果发生IO事件后还能够检测到，但是当检测到IO事件和poll（）就不会再超时了，就会一直等待监听事件的发生。

这一点和select（）有所不同，select（）超时后就不在监听IO事件的发生了，而是一直返回超时。

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/time.h>
#include <signal.h>
#include <poll.h>

int main(void)
{
    int fd=-2;
    char buf[20];
    struct pollfd pfd[2]={0};  //监视两个文件句柄所以定义了一个结构体数组

    bzero(buf,sizeof(buf));
    fd=open("/dev/input/mouse0",O_RDONLY);
    if (fd<0)
    {
        perror("open:");
    }
    printf("already read\n");
    pfd[0].fd=0;           //监视的文件描述符
    pfd[0].events=POLLIN;  //设置为监视读取事件
    pfd[1].fd=fd;
    pfd[1].events=POLLIN; 
    while (1)
    {
        int ret=poll(pfd,fd+1,1000);
        if (ret<0)
        {
            perror("why");
        }
        else if (ret==0)
        {
            printf("等待超时\n");
        }
        else
        {
            if(pfd[0].events==pfd[0].revents) //判断是否为监视的事件发生
            {
                bzero(buf,sizeof(buf));
                read(0,buf,5);
                printf("read keyboard data=%s\n",buf);
            }
          
          //此处为轮询句柄数组的每一个值
          
            if (pfd[1].events==pfd[1].revents)
            {
                bzero(buf,sizeof(buf));
                read(fd,buf,5);
                printf("read mouse data=%s\n",buf);
            }
        }
    }
    return 0;
}
```

# 异步IO

异步IO相当于系统内核为我们实现的一套软件中断程序，如下代码中的function（）函数就可以比喻为中断服务函数。

主程序在完成一些异步IO的初始化之后，在while（）循环中检测键盘的输入，

但是如果注册了异步事件的IO事件发生，就会去执行异步任务函数，即获取鼠标输入的数据。

## 代码演示

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <signal.h>
int fd=0;
void function(int sig)
{
    char buf[5]={0};
    if (sig!=SIGIO)
    {
        return;
    }
    else
    {
        read(fd,buf,sizeof(buf));
        printf("read mouse data=%s\n",buf);
    }
}

int main(void)
{
    char buf[20];
    int flag=-1;
    bzero(buf,sizeof(buf));
    fd=open("/dev/input/mouse0",O_RDONLY);
    if (fd<0)
    {
        perror("open:");
    }
    printf("already read\n");
    //把鼠标的文件描述符设置为可以接收异步IO
    flag=fcntl(fd,F_GETFL);
    flag|=O_ASYNC;
    fcntl(fd,F_SETFL,flag);
    //把异步IO事件的接收进程设置为当前进程
    fcntl(fd,F_SETOWN,getpid());
    //注册当前进程的SINGIO信号捕获函数
  //利用了信号量
    signal(SIGIO,function);
    while (1)
    {
        memset(buf,0,sizeof(buf));
        read(0,buf,sizeof(buf));
        printf("read keynoard =%s\n",buf);
    }
    return 0;
}
```

