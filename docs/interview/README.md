> 等到鸡啄完了米，狗舔完了面，火烧断了锁，一些人可能就会发现这些问题在现实中完全没有用

- 都是简单的概念性答案，并没有什么逻辑性可言，当作死记硬背的填空题使用
- 仅针对不熟悉或是不需要在实际工作中去深入了解的知识，相信对于在真实工作中使用的也不需要在此列出
# Network
## TCP vs UDP
- 是否面向连接
- 是否可靠传输
- stateful？
- 传输效率

## TCP三次握手、四次挥手
1. client -> server SYN(SEQ=x) client进入SYN_SEND状态，等待server确认
2. server -> client SYN + ACK(SEQ=y, ACK=x+1) server进入SYN_RECV状态
3. client -> server ACK(ACK=y+1) client and server 进入ESTABLISHED状态
- 第一次握手server确认对方发送正常 + 自己接收正常
- 第二次握手client确认自己发送 + 接收正常，对方发送 + 接收正常， server确认对方发送 + 自己接收正常
- 第三次握手：server确认自己发送 + 接收正常， 对方发送 + 接收正常
1. client -> server FIN(SEQ=x) client进入FIN-WAIT-1状态
2. server -> client ACK(ACK=x+1) server进入CLOSE-WAIT状态，client进入FIN-WAIT-2状态
3. server -> client FIN(SEQ=y) 服务端进入LAST-ACK状态
4. client -> server ACK(ACK=y+1) client进入TIME-WAIT状态， server进入CLOSE状态，client在2MSL后仍没有收到server回复，证明server正常关闭，随后client自己也关闭连接
- TCP是全双工通信，双向都可以传输数据，发出连接释放通知以后，需要对方确认也没有数据再发送

# OS
## 线程间同步方式
- 互斥锁
- 读写锁：多读单写
- Semaphore：允许同一时刻多线程访问同一资源，但需要控制最大线程数量
- Barrier：等待多个线程达到某个点再一起继续执行
- Event：通过通知操作的方式保持多线程同步

## IPC 进程间通信
- pipes and Named pipes
- signal
- message queuing
- semaphores
- shared memory
- sockets

## 常见进程调度算法
- first come, first served
- shortest job first
- round-robin
- priority
- multi-level feedback queue

## 僵尸进程和孤儿进程
- 僵尸进程：子进程终止，但父进程仍在运行，且没有释放子进程的资源占用
- 孤儿进程：进程仍在运行，但其父进程已经终止或不存在

## 内存管理主要做了什么
- 内存分配和回收
- 地址转换
- 内存扩充
- 内存映射
- 内存优化
- 内存安全

## 内存碎片
内存碎片由内存的申请和释放产生
- 内部内存碎片：已分配给进程但未被使用
- 外部内存碎片：为分配的连续内存太小不能满足任意进程所需要的内存分配请求

## 虚拟内存
作为进程访问主存（物理内存）的桥梁并简化内存管理
- 隔离进程
- 提升物理内存利用率
- 简化内存管理
- 共享物理内存
- 更大的可使用内存空间

## 虚拟地址和物理地址
虚拟地址翻译成物理地址的主要机制
- 分段：以一段连续的物理内存的形式管理和分配物理内存
  - 通过段表映射虚拟地址和物理地址
  - 段号 + 段内偏移量 -> 最终物理地址
  - 会导致内存外部碎片
- 分页：把主存分为连续等长的物理页，虚拟地址空间也分为连续等长的虚拟页
  - 离散分配：任意虚拟页可以被映射到任意物理页
  - 通过页表映射虚拟地址和物理地址，虚拟地址由页号 + 页内偏移量组成
  - 多级页表：解决单级页表浪费空间问题，每个页表和前一个页表相关联，时间换空间
  - 换页机制：当物理内存不够的时候，将一些物理页的内容放到磁盘上去
- 段页：把物理内存先分成若干段，每个段再继续分成若干大小相等的页

## 软链接和硬链接
文件链接是一种特殊的文件类型，可以在文件系统中指向另一个文件
- hard link: `ln` 命令创建
  - 通过唯一的索引节点inode号建立连接，删除一个对另外一个没有影响
  - 只有删除源文件和所有硬链接文件，该文件才会被删除
- symbolic link： 指向文件路径， `ln -s` 创建软链接
  - 源文件删除后软链接依然存在，但指向的是一个无效路径

## 常见磁盘调度算法
- first come fist served
- shortest seek time first
- scan/circular scan
- look/c-look

# Mysql
## 关系型数据库和非关系型数据库
- relational database 使用表来存储数据，no-relational database 使用不同的数据模型，文档、kv、图

## 索引
用于快速查询和检索数据的数据结构，其本质可以看成排序好的数据结构，mysql使用B+树作为索引结构
- 加快检索速度，唯一性索引可以保证每一行数据的唯一性
- 创建和维护索引有额外开销，索引需要物理文件存储，数据量不大时索引不一定能带来很大提升
- 主键使用的就是主键索引
- 选择合适的字段创建索引
  - 不为null
  - 被频繁查询的字段
  - 被作为条件查询的字段
  - 频繁需要排序的字段
  - 频繁用于连接的字段
- 被频繁更新的字段要慎重建立索引
- 每张表的上线索引数量不超过5个