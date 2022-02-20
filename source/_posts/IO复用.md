---
 title: IO复用
date: 2022-02-20 09:34:28
tags: Linux
categories: Linux网络编程
---
# Select

```C++
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

优点：跨平台

缺点：监听上限受文件描述符限制，最大1024

需要检测满足条件的fd，只能通过轮询和检查来判断事件，而且只能支持三种事件

# poll



```C++
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
struct pollfd {
    int  fd;     /* file descriptor */
    short events;   /* requested events */
    short revents;  /* returned events */
};
```

传入一个struct pollfd数组，指定其大小。结构体中包含文件描述符、传入的监听事件和传出的已触发事件。时间都由宏来定义。返回值是有多少fd有事件。

需要做轮询，将revent与特定事件做与运算，来判断事件是否发生。

优点：自带数据结构，可以拓展监听上限，不像select需要结束fd_set（无法修改数组长度，只能1024，内核限制）；可以将监听事件集合和返回事件集合分离。

缺点：专属Linux，不能跨平台；无法直接满足监听事件的文件描述符。

 

## 修改监听fd的1024上限

 `cat /proc/sys/fs/file-max`这个命令可以查看当前计算机所能打开的最大文件个数。受硬件限制。

`ulimit -a`查看当前用户下的进程，默认打开文件描述符的个数。默认为1024。

`sudo vi /etc/security/limits.comf`来修改，软上限可以用命令修改，不能超过硬上限。硬上限只能在文件中修改。修改后需要重新登录来生效。

# epoll

```C++
int epoll_create(int size);
int epoll_create1(int flags);
```

返回值是一个epoll文件描述符（句柄），指向一个红黑树的根节点，每个节点都是一个要监听的fd。

size是预计需要监听的节点数量，仅供内核参考，由内核创建红黑树，可以动态调整。

```C++
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```

- `epfd`：epoll文件描述符
- `op`：操作，增改删，`EPOLL_CTL_ADD`、`EPOLL_CTL_MOD`、`EPOLL_CTL_DEL`
- `fd`：待监听的fd
- `event`：`struct epoll_event`的指针，**传出参数**。如果fd有事件，这个参数就会被设置。
  - `events`：`EPOLLIN`、`EPOLLOUT`、`EPOLLERR`
  - `data`：`union`类型，`ptr`是泛型指针，可以是自定义的结构体（里面包含回调函数），`fd`是对应监听事件。

```C++
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
```

- `events`：注意，`events`表示这是个数组，用来保存内核得到的事件的集合。**传出参数**，传出满足监听条件的fd。
- `maxevents`：数组元素的总个数，`events`的长度。

返回满足监听的总个数，可以用作循环上限。`events`一定都是满足监听的事件，可以直接循环处理。

适合使用场景：大量并发连接，但是只有少量活跃连接。

## 事件模型

epoll有两种事件模型，**默认是水平触发**。修改为边缘触发则在`epoll_ctl`的参数`event.events`中用或运算加上`EPOLLET`。

1. 边缘触发（Edge Triggered，ET）

2. 水平触发（Level Triggered，LT）

用通信的知识来理解，假如只有电压发生变化才会产生触发，那么就是边缘触发。如果电压维持高水平，但是依然会触发，那么就是水平触发。

epoll在处理事件时，如果没有将缓冲区读取完，或者在处理过程中产生了新的触发，在ET模式下是不会触发的，只有下一次触发时再继续处理。

LT模式可应用与阻塞和非阻塞的文件描述符。ET是**高速模式**，只能用于非阻塞（给fd设置`O_NONBLOCK`）。

缺点：不能跨平台，只支持Linux（Unix也不行）

## epoll反应堆模型

ET模式+非阻塞模式+回调函数`void* ptr`

