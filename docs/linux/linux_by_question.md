# epoll
一种高性能事件通知机制

几个关键概念：
- File Descriptors: 几乎所有的I/O操作都通过文件描述符，epoll监视的是文件描述符上的事件
- epoll实例：一个内核数据结构，红黑树，链表
- 事件类型：
  - EPOLLIN
  - EPOLLOUT
  - EPOLLERR
- `epoll_ctl`: 向epoll注册文件描述符和事件类型，以监听特定事件
- `epoll_wait`: 等待文件描述符上的事件发生，事件发生时会返回fd信息
- 
原理简述：
- 在内核中通过一个双向链表来管理待监视的fd,通过红黑树来管理已就绪的fd
- fd上发生事件时，内核将其添加到就绪链表，通过`epoll_wait`返回给用户空间，`epoll_wait`会阻塞程序