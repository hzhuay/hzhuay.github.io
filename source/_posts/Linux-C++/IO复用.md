---
title: IO复用
date: 2022-02-20 09:34:28
tags: Linux
categories: C++
---
## Select

```C++
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

返回值：就绪（可读、可写、异常）的文件描述符的总数。

- `nfds`是指定被监听的文件描述符的总数，通常被设置为被监听的所有文件描述符中的最大值+1，因文件描述符从0开始计算。也就是说其实这个参数不一定代表数量。

- 后三个参数分别对应可读、可写、异常三种事件的文件描述符集合，类型为`fd_set`，内部是一个数组，总bit位受到`FD_SETSIZE`限制，有上限。这个数组的每一位表示对某个fd监听特定类型的事件。假设现在fd_set的数组一共8bit，我们要监听fd=3的读事件，那么readfds内的数组为0000,1000。

    注意，这三个参数即是传入参数，也是传出参数。传入时表达用户希望监听的fd，穿出时表达就绪的fd。确实有发生对应事件的fd对应位置还是1，没有事件的bit位会被清0。

- `timeout`等待时间，null为阻塞等待。

网络程序中，select能处理的异常情况只有一种：socket上接收到带外数据。

**什么是带外数据**？

带外数据(out—of—band data)，有时也称为加速数据(expedited data)，是指连接双方中的一方发生重要事情，想要迅速地通知对方。这种通知在已经排队等待发送的任何“普通”(有时称为“带内”)数据之前发送。

带外数据设计为比普通数据有更高的优先级。

带外数据是映射到现有的连接中的，而不是在客户机和服务器间再用一个连接。

socket收到普通数据和带外数据都将使select返回，但是前者处于可读状态，后者处于异常装填。

以下是一个典型的基于select的服务端：

```C++
while(1){//这两部是非常重要的，不可缺少的，缺少了有可能导致状态不会更新
  FD_ZERO(&fdsr); //初始清0
  FD_SET(sock_fd, &fdsr); //将监听连接fd加入集合

  //将所有的连接全部加到这个这个集合中，可以监测客户端是否有数据到来
  for(i = 0; i < MAXCLINE; i++) FD_SET(fd[i],&fdsr);

  //如果文件描述符中有连接请求 会做相应的处理，实现I/O的复用 多用户的连接通讯
  ret = select(maxsock + 1,&fdsr,NULL,NULL,&tv);
  if(ret <0) break;//没有找到有效的连接 失败
  else if(ret ==0) continue;// 指定的时间到，
  //下面这个循环是非常必要的，因为你并不知道是哪个连接发过来的数据，所以只有一个一个去找。
  for(i=0;i<conn_amount;i++){
		if(FD_ISSET(fd[i],&fdsr))	{
      ret = recv(fd[i],buf,sizeof(buf),0);
      //如果客户端主动断开连接，会进行四次挥手，会出发一个信号，此时相应的套接字会有数据返回，告诉select，我的客户断开了，你返回-1

      if(ret <=0){ //客户端连接关闭，清除文件描述符集中的相应的位
      	close(fd[i]); FD_CLR(fd[i],&fdsr);
      	fd[i]=0; conn_amount--;
      }else {}//否则有相应的数据发送过来 ，进行相应的处理
    }
		if(FD_ISSET(sock_fd,&fdsr))//有用户连接
      new_fd = accept(sock_fd,(struct sockaddr *)&client_addr,&sin_size);
		
    //添加新的fd 到数组中 判断有效的连接数是否小于最大的连接数，如果小于的话，就把新的连接套接字加入集合
    if(conn_amount < MAXCLINE){
			for(i = 0; i < MAXCLINE; i++){
				if(fd[i]==0){
					fd[i] = new_fd;
					break;
				}
			}
			conn_amount++;
			if(new_fd > maxsock) maxsock = new_fd;
		}else{ // 超过连接数量
			close(new_fd);
			continue;
		}
  }
}
```

优点：跨平台

缺点：

- 监听上限受文件描述符限制，最大1024（这个因机器而异）

- 需要检测满足条件的fd，只能通过轮询和检查来判断事件，而且只能支持三种事件
- 每次监听fd的标志会被清0（如果没有事件就绪），需要重新设置

## poll

```C++
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
struct pollfd {
    int  fd;     /* file descriptor */
    short events;   /* requested events */
    short revents;  /* returned events */
};
```

传入一个struct pollfd数组，指定其大小。结构体中包含文件描述符、传入的监听事件和传出的已触发事件。`nfds`指定被监听事件集合`fds`的大小，`timeout`指定超时值，单位是毫秒，-1时阻塞，0时立刻返回。

返回值是有多少fd有事件。

需要做轮询，将revent与特定事件做与运算，来判断事件是否发生。

以下是一个典型服务端：

```C++
// 设置监听连接的soclet
fds[0].fd = listenfd;
fds[0].events = POLLIN | POLLERR;
fds[0].revents = 0;

while( 1 ){
  ret = poll( fds, user_counter+1, -1 );
  if ( ret < 0 ) break;

  for( int i = 0; i < user_counter+1; ++i ){
    if( ( fds[i].fd == listenfd ) && ( fds[i].revents & POLLIN ) ){ // 如果是有新用户连接
      struct sockaddr_in client_address;
      socklen_t client_addrlength = sizeof( client_address );
      int connfd = accept( listenfd, ( struct sockaddr* )&client_address, &client_addrlength );
      if ( connfd < 0 ) continue;
      if( user_counter >= USER_LIMIT ){}//用户连接数超出上限
      user_counter++;
      // 设置新用户的socket属性，加入fds数组
      users[connfd].address = client_address;
      setnonblocking( connfd );
      fds[user_counter].fd = connfd;
      fds[user_counter].events = POLLIN | POLLRDHUP | POLLERR;
      fds[user_counter].revents = 0;
    }else if( fds[i].revents & POLLERR ){}//遇到异常
    else if( fds[i].revents & POLLRDHUP ){//用户断连
      users[fds[i].fd] = users[fds[user_counter].fd];
      close( fds[i].fd );
      fds[i] = fds[user_counter];
      i--; user_counter--;
      printf( "a client left\n" );
    }else if( fds[i].revents & POLLIN ){//用户发送数据到服务端
      int connfd = fds[i].fd;
      memset( users[connfd].buf, '\0', BUFFER_SIZE );
      ret = recv( connfd, users[connfd].buf, BUFFER_SIZE-1, 0 );
      printf( "get %d bytes of client data %s from %d\n", ret, users[connfd].buf, connfd );
      if( ret < 0 ){
        if( errno != EAGAIN ){}//出错，与用户断连，关闭socket
      }
      else if( ret == 0 ){}//理论上不会到达这个分支
      else{
        for( int j = 1; j <= user_counter; ++j ){
          if( fds[j].fd == connfd )continue;
          //这段代码怪怪的
          fds[j].events |= ~POLLIN;//这一步之后读状态不变，其他全部为1
          fds[j].events |= POLLOUT;//在上一步基础上加上写事件，其他全部不变
          users[fds[j].fd].write_buf = users[connfd].buf;
        }
      }
    }
    else if( fds[i].revents & POLLOUT ){//用户有写事件，表示服务器需要向该客户发送数据
      int connfd = fds[i].fd;
      if( ! users[connfd].write_buf )continue;
      ret = send( connfd, users[connfd].write_buf, strlen( users[connfd].write_buf ), 0 );
      users[connfd].write_buf = NULL;
      fds[i].events |= ~POLLOUT;
      fds[i].events |= POLLIN;
    }
  }
}
```

优点：自带数据结构，可以拓展监听上限，不像select需要结束fd_set（无法修改数组长度，只能1024，内核限制）；可以将监听事件集合和返回事件集合分离。

缺点：专属Linux，不能跨平台；无法直接满足监听事件的文件描述符。

 

### 修改监听fd的1024上限

 `cat /proc/sys/fs/file-max`这个命令可以查看当前计算机所能打开的最大文件个数。受硬件限制。

`ulimit -a`查看当前用户下的进程，默认打开文件描述符的个数。默认为1024。

`sudo vi /etc/security/limits.comf`来修改，软上限可以用命令修改，不能超过硬上限。硬上限只能在文件中修改。修改后需要重新登录来生效。

## epoll

Linux特有的IO复用函数。

实现和使用与select和poll有很大差异。首先，epoll使用一组函数来完成任务。其次，epoll把用户关心的文件描述符上的事件放在内核里的一个事件表中，从而避免像select和poll那样每次调用都要重复传入文件描述符集或事件集。但epoll需要使用额外一个文件描述符，来唯一标识内核中的事件表。

```C++
int epoll_create(int size);
int epoll_create1(int flags);返回值是一个epoll文件描述符（句柄），指向一个红黑树的根节点，每个节点都是一个要监听的fd。
```

size是预计需要监听的节点数量，仅供内核参考，由内核创建红黑树，可以动态调整。

```C++
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
typedef union epoll_data {
    void        *ptr;
    int          fd; //联合体中fd用得最多
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;
```

该函数用来操作epoll的内核事件表。

- `epfd`：epoll文件描述符
- `op`：操作，增改删，`EPOLL_CTL_ADD`、`EPOLL_CTL_MOD`、`EPOLL_CTL_DEL`
- `fd`：待监听的fd
- `event`：`struct epoll_event`的指针。
  - `events`：`EPOLLIN`、`EPOLLOUT`、`EPOLLERR`
  - `data`：`union`类型，`ptr`是泛型指针，可以是自定义的结构体（里面包含回调函数），`fd`是对应监听事件。

```C++
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
```

主要接口，它在一段超过时间内等待一组文件描述符上的事件。

- `events`：注意，`events`表示这是个数组，用来保存内核得到的事件的集合。**传出参数**，传出满足监听条件的fd。
- `maxevents`：数组元素的总个数，`events`的长度。

返回满足监听的总个数，可以用作循环上限。`events`一定都是满足监听的事件，可以直接循环处理。与poll的对比：

```C++
//如何索引poll返回的就绪文件描述符
int ret = poll(fds, MAX_EVENT_NUMBER, -1);
//必须遍历所有已注册文件描述符并找到的其中的就绪者
for(int i = 0; i > MAX_EVENT_NUMBER; ++i){
  if(fds[i].revents & POLLIN){
    int sockfd = fds[i].fd;
    //处理事件
  }
}

//如何索引epoll返回的就绪文件描述符
int ret = epoll_wait(epollfd, events, MAX_EVENTS_NUMBER, -1);
//仅遍历就绪的ret个文件描述符
for(int i = 0; i < ret; i++){
  int sockfd = events[i].data.fd;
  //直接处理事件，不需要再判断是否就绪
}
```

适合使用场景：大量并发连接，但是只有少量活跃连接。

### 事件模型

epoll有两种事件模型，**默认是水平触发**。修改为边缘触发则在`epoll_ctl`的参数`event.events`中用或运算加上`EPOLLET`。

1. 边缘触发（Edge Triggered，ET）：epoll在处理事件时，如果没有将缓冲区读取完，或者在处理过程中产生了新的触发，在ET模式下是不会触发的，只有下一次触发时再继续处理。

2. 水平触发（Level Triggered，LT）

用通信的知识来理解，假如只有电压发生变化才会产生触发，那么就是边缘触发。如果电压维持高水平，但是依然会触发，那么就是水平触发。

LT模式可应用与阻塞和非阻塞的文件描述符。ET是**高速模式**，只能用于非阻塞（给fd设置`O_NONBLOCK`）。

缺点：不能跨平台，只支持Linux（Unix也不行）

### epoll反应堆模型

ET模式+非阻塞模式+回调函数`void* ptr`

