Interview preparation for [About Me](about/README.md)

# 复盘

## 20230710 APITable
1. 写一个函数，该函数接收一个字符串，将该字符串中连续的空格合并为一个，并返回新的字符串
```rust
fn merge_spaces(input: &str) -> String {
    let mut res = String::new();
    let mut prev_char: Option<char> = None;

    for ch in input.chars() {
        match (prev_char, ch) {
            (Some(prev), ' ') if prev == ' ' => continue,
            _ => res.push(ch),
        }
        prev_char = Some(ch);
    }
    res
}
```
上为正解，当时只想到粗暴的for循环 + count计数，还是不够rusty,想不到从match, (a, b)入手

2. `trait Iterator`的定义，以及用rust写一个函数，该函数返回一个迭代器，该迭代器返回斐波那契数列
```rust
fn fib() -> Box<dyn Iterator<Item = u64>> {
    let mut current = 0;
    let mut next = 1;
    Box::new(
        std::iter::from_fn(move || {
            let new_next = current + next;
            let new_current = std::mem::replace(&mut next, new_next);
            current = new_current;
            Some(new_current)
        })
    )
}
```
看得出来想问的是`trait object`，所以就往这个方向答了，但不通过具体一个`struct`去`impl`要用到`std::iter::from_fn(f: F) -> FromFn<F> where F: FnMut() -> Option<T>`，之前没用过。没有具体实现，讲了一下思路，问除了u64还有什么限制，没想到

还问了`Iterator`为什么不用泛型而是关联类型，这个问题忘记在哪里看到过，主要答了泛型的传染性。在灵活性、兼容性、扩展性上都有优势，同时可以简化类型参数，提高可读性

3. Java问的比较简单，可能不太了解信令侧的东西。
   1. 为什么用netty？因为希望是无状态的服务器，不需要很复杂的实现，只想要一个简单的框架帮助构建好channel通道（瞎扯）
   2. 为什么要用springboot启动？需要提供一些http接口对外
   3. 自动化运维指什么？jenkins + ansible等
4. 最后问了最近有什么挑战
5. 日常反问：问了rust将来是否会换掉java

其实问的都不难，但还是没答好。面试之前要填问卷，拉代码本地部署，成本还是有点高的

# 开发

## Proxy
1. IP fragmentation
    - MTU: Maximum Transmission Unit.the largest packet a data link layer can send, including its header.
    - Performed by the sender or the forwarding routers
    - 相关字段: Identification, Fragment offset, Flags
    - steps
        1. 为每一个分片的包copy header + payload
        2. reset flags: 除了最后一个被分片的包被设置为DF（000），其余被分片的包设置为MF（001）
        3. reset fragment offset: 第一个包为0,此外除了最后一个包其余所有包都为8的倍数，具体计算方法见下面代码: offset/8
        4. reset identification: 被分片的包应该具有相同的id
        5. reset total length: 切片了原来的数据包，自然要重新计算包长度
        6. reset checksum: 同上
    - The reassembly process is carried out at the destination

```java
for (int i = 0; i < fragmentCount; i++) {
            IPPacket packet = new IPPacket(1);
            byte[] fragment;
            if (totalPayload - fragmentMtu > 0) {
                fragment = new byte[fragmentMtu + 20];
            } else {
                fragment = new byte[totalPayload + 20];
            }

            //copy IP header
            System.arraycopy(source, 0, fragment, 0, 20);
            //copy payload
            System.arraycopy(source, 20 + offset, fragment, 20, fragment.length - 20);

            packet.setData(fragment);
            //1. reset mf
            if (i == fragmentCount - 1) {
                packet.setIPFlags(000);
            } else {
                packet.setIPFlags(001);
            }
            //2. reset fragment offset
            packet.setFragmentOffset(offset / 8);
            //3. reset identification
            packet.setIdentification(testId);
            //4. reset total length
            packet.setIPPacketLength(fragment.length);
            //5. recalculate checksum
            packet.computeIPChecksum(true);

            courier.send(desIp, fragment);

            offset += fragment.length - 20;
            totalPayload -= fragmentMtu;
        }
```
2. 为什么要用到raw socket
   - raw sockets allows you to bypass the TCP/UDP layer (Layer 4) in the RT-TCP/IP stack and communicate directly with the Network IP layer (Layer 3). 

3. routing design
    - 基于规则和标签的路由设计：
        1. 每个号码入库时贴上能力标签
        2. 命中号码时根据当前通话为每个号码打上tag
        3. 按优先级遍历规则表，命中规则后结束遍历，拿到规则id，如过对应规则为多节点，则经过负载均衡算法后选择一个目标地址进行分发
    - 规则通过遍历条件，正则判断是否满足，一个规则可以有多个条件，条件之间默认and关系
    - 负载均衡算法：Smooth Weighted Round-Robin Balancing
    ```java
    private static Track next(List<Track> list) {
        if (list.size() == 1) {
            return list.get(0);
        }

        int sum = sum(list);
        // record currentWeight and compare to next one
        int temp = Integer.MIN_VALUE;
        //record the index of the track which has the largest currentWeight in list
        int index = Integer.MIN_VALUE;
        for (Track track : list) {
            // 1. currentWeight += weight
            track.setCurrentWeight(track.getCurrentWeight() + track.getWeight());
            // 2. select peer
            if (track.getCurrentWeight() > temp) {
                temp = track.getCurrentWeight();
                index = list.indexOf(track);
            }
        }
        // 3. currentWeight -= sum
        Track track = list.get(index);
        track.setCurrentWeight(track.getCurrentWeight() - sum);
        return track;
    }
    ```
4. heartbeat detecting
    - 对每个非测试节点发送OPTIONS, 超过3s没有收到回复的视为dead node, 将其权重降为0, 不再分发信令，同时改变该节点的状态，同理，当dead node重新回复200OK, 恢复该节点权重
    ```java
    @Data
    private class HeartbeatTimer {
        private String host;
        private int port;
        private String key;
        private Stopwatch clock;
        private boolean alive;

        private HeartbeatTimer(String host, int port) {
            this.host = host;
            this.port = port;
            this.key = host + ":" + port;
            clock = Stopwatch.createStarted();
            selfCheck();
        }

        private void selfCheck() {
            Runnable clockCheck = (() -> {
                if (clock.elapsed().getSeconds() >= config.getHeartbeat().getTimeoutInterval()) {
                    alive = false;
                    clearWeight(host, port);
                }
            });
            service.scheduleAtFixedRate(clockCheck, (long) config.getHeartbeat().getOptionsDelay() + 1, 1, TimeUnit.SECONDS);
        }

    } 
    ```
## OAM Web
1. 信令流程图：抓取不同机器的信令包，解析为更加可读的流程图形式，主要用gojs实现
2. 中间号CRUD：提供对号码增删改查基本操作的web入口
3. proxy分流：对规则表、条件表提供crud入口，以及对节点的各种管理（权重调整、手动分流等）
4. 媒体配置流程图：拖拽形式的媒体流程配置图，实现在web上改变通话逻辑（通过前端配置媒体服务器）

## SIP MVC
1. SpringMVC
   1. DispatchServlet接收来自用户的请求，调用HandlerMapping得到Handler
   2. DispatchServlet调用HandlerAdapter执行Handler得到ModelView
   3. DispatchServlet将ModelView传给ViewResolver解析得到View
   4. DispatchServlet对View进行渲染后返回用户
2. SIP MVC
   1. DefaultSipListener接收sip消息（request/response），将消息传递给interceptor链
   2. interceptor链为每一条消息初始化上下文、状态
   3. interceptors按照sip协议，针对不同的消息找到Controller下对应的Handler，更新state
   4. Handler根据不同的消息，注入所需的参数，完成业务处理，改变状态
3. 对应关系
   1. DispatchServlet <--> DefaultSipListener
   2. HandlerMapping <--> Interceptors
   3. Handler <--> Handler


# 运维
1. 运维、监控体系搭建
   - 中间件搭建：nacos, redis, etcd...
   - 业务监控接入告警：prometheus + grafana
   - 各种组件漏洞修复，运维接口工作（工单、上线）

2. 自动化运维：jenkins、ansible
   - go组件、java组件、c++组件的jenkins + ansible自动部署脚本
   - 维护go组件、c++组件docker编译和打包环境


# Rust
1. Bevy, Axum

Axum is the Bevy of web frameworks. [Source](https://www.reddit.com/r/rust/comments/12n1vdc/rust_axum_full_course_web_development/)

- In Bevy, you use/define Components and Resources (the parts) that you can ingest in your System functions (the "processors").

- In Axum, you use/define Extractors and States (the parts) that you can ingest in your Handlers/Middleware (the "processors").

# 其他

0. 自我介绍
    1. 面试官好，我是姜子豪，今年24岁，中山大学电信院2021级本科毕业生，目前就职于中移互联网，岗位是后端研发，在组内的定位是一名运维开发。
    2. 开发方面我主要负责信令侧组件开发和运维网站的搭建，信令侧开发包括proxy代理服务器、as应用服务器，运维网站是面向团队内部使用的web服务
    3. 运维方面主要负责三个部分，一个是运维、监控体系的中间件搭建和告警配置，一个是自动化运维的部署实现，负责维护团队内go组件、c++组件和java组件的docker编译/打包环境，结合jenkins+ansible实现自动部署，最后是负责主机和各种第三方组件的漏洞修复


1. 项目：实现方法 -> 寻找你项目的问题 -> 未解决的问题，是否有方案？
2. 近期有没有遇到什么挑战？
   - sipmvc可配置化的实现：原有jsip框架处理消息实际上是同步的，希望改为异步。比如routine A收到信令后发消息给routine B，要求routine B处理完该消息后通知routine A，A才释放该信令。B收到A发来的消息应该由线程池（sipmvc自己创建的）分配线程并执行routine B（由消息唤醒而非信令）处理消息

# APITable二面(boss)
databus定位是数据组件

1. DataBus definition: DataBus提供了一种解耦的方式，使得不同的组件可以独立开发、部署和扩展，同时能够通过数据总线进行数据交换。它可以处理不同组件之间的异步通信，从而提高系统的可伸缩性和可靠性。
2. OT: Operational Transformation, 是一种用于实现实时协同编辑的算法。它旨在解决多个用户并发编辑同一文档时可能发生的冲突和不一致性问题
   1. 转换函数：每个操作都有一个相应的转换函数。转换函数根据操作的作用位置和其他操作的状态，将一个操作转换为另一个操作。这是为了确保在并发编辑时，操作可以以一致的方式应用于文档。
   2. 转换过程：当两个或多个用户同时编辑同一文档时，他们的操作可能会冲突。为了解决冲突，OT算法将每个用户的操作转换为其他用户的操作。这个转换过程包括应用操作、转换操作和发送操作。
3. Why DataBus in rust?
   1. DataBus是AI的关键构成
   2. 实时的协同数据流太分散和凌乱，导致迭代功能困难
   3. 消除重复代码
   4. rust的性能优势
4. DataBus分层
   1. Application
   2. BO(Business logic)/DataObjectManager: 面向对象的方式抽象封装space, DataSheet, View...
   3. Facades: 各种语言的Binding
   4. SO(Domain logic)/DataFunctionManager, 纯函数
   5. DAO, 持久化层
   6. OT
   7. Shared：工具类
5. 怎么理解DataBus：处理数据流，向上提供bindings以及各种接口（较高层级的抽象和封装，屏蔽掉复杂的协作流程），向下处理DAO和OT
6. DataBus有什么好处：
   1. 将数据层与其他业务层隔离解耦和，不同的业务组件可以独立开发部署，通过DataBus进行数据的交换
   2. DataBus提升性能，集中优化与数据有关的操作，在此层处理不同组件间的通信，比如异步
   3. 只提供顶层的接口，意味着下层的数据库被解放，可以支持多数据源，同时OT的处理也在此层被屏蔽
   4. 统一数据相关的函数/接口，消除冗余代码
   
# 优维
1. docker/k8s
   1. namespaces：分离进程树、网络接口、挂载点以进程间通信等资源的方法，比如Docker内部对宿主机进程一无所知，就是使用了`CLONE_NEWPID`namespace
   2. Docker run启动的容器都具有单独的网络命名空间，默认为Bridge模式。该模式下，Docker会为所有容器设置IP地址，Docker服务启动后会创建虚拟网桥docker0，所有服务默认与该网桥相连
      1. 容器创建时会创建一对虚拟网卡，其中一个放到容器中，加入到docker0的网桥
      2. docker0为每一个容器分配一个新的IP地址，docker0的IP地址设置为默认网关
      3. docker0通过iptables与宿主机网卡相连，所有符合条件的请求会通过iptables转发到docker0,再分发给对应的机器
      4. Docker通过Linux命名空间实现网络隔离，又通过iptables数据包转发，实现为宿主机器或容器提供服务
   3. 挂载点：使用`CLONE_NEWS`实现子进程得到父进程挂载点的拷贝，否则子进程对文件的读写会同步回父进程以及整个主机的文件系统；还需要通过`privot_root`/`chroot`改变进程能够访问文件目录的根节点
   4. CGoups: 隔离宿主机上的物理资源，比如CPU和内存
   5. **Linux命名空间解决了进程、网络以及文件系统的隔离，控制组解决了CPU、内存等资源的隔离**
   6. Docker镜像本质就是一个文件/压缩包
   7. Docker每一个镜像都是由一系列只读的层组成，Dockerfile每一个命令会在已有的只读层上创建一个新的层
   8. 每一个容器等于镜像加上一个可读写的层，所以同一个镜像可以对应多个容器
   9. AUFS：联合文件系统，Docker会通过联合的方式组装镜像层，所有镜像层只读，只有最顶层可以被用户读写
2. linux/network programming
3. tcp/ip
   
# 美的
1. netty/tokio/async
   1. why sync?
      - 线程开销大：切换耗时，包括内核调度开销，污染cpu的cache
      - 线程保留的栈占空间
   2. 分类
      - 提供future，有回调地域问题，开发成本和维护成本提高，所以会套一层async/await,上层同步，底层异步
      - CSP,协程，sackful,有一定开销，且语言写死无法更改
   3. netty
      - 以io resource为入口，而不是task
      - 所有的读写都是别人回调你，而不是你主动主动调用
      - 使用NIO和多线程
      - pipeline并不适合所用场景
   4. tokio
      - async/await：表面同步内在异步，减少回调地狱的问题
      - 以task为单位，tokio::spawn是轻量级线程（green thread）

2. Java21
   1. Virtual Threads
   2. Sequenced Collection
   3. 解构
   4. switch增强
   5. 分代式ZGC Generational ZGC
   6. `_`表示未命名变量
   7. JDK Foreign Function

3. ZGC
   - 特点和优势
     1. 停顿时间短
     2. 并发收集
     3. 大堆支持
     4. 高吞吐量
     5. 自适应
   - 核心原理
     - 通过并发执行垃圾回收操作来减少STW时间
     - 并发的标记整理: 并发意味着GC线程和应用线程都在不停的访问对象，这可能导致对象发生转移以后地址未更新，应用线程访问到旧地址。在ZGC中应用线程访问对象会触发“读屏障”，会把读出来的地址更新到新地址，JVM通过着色指针来判断对象被移动过
   - 触发机制：
     - 基于分配速率的自适应算法，ZGC根据近期对象分配速率以及GC时间计算当内存占用到达什么阈值时触发下一次GC
     - 基于固定时间间隔
     - 外部触发（`System.gc()`）
  
# Rust
## IPC(InterProcess Communication)
任何一个进程的全局变量在另一个进程中都看不到，所以进程之间要交换数据必须通过内核：在内核中开辟一块缓冲区，进程1把数据从用户空间拷贝到缓冲区，进程2再从内核缓冲区把数据读走
1. 管道/匿名管道（Pipe）
   - 管道是半双工，数据只能向一个方向流动
   - 只能用于父子进程或者兄弟进程
   - 管道单独构成一种独立的文件系统：管道对于两端的进程而言就是一个文件，且只存在于内存中
   - 本质是一个有限的内核缓冲区，FIFO,可以看作一个循环队列
2. 有名管道
   - 主要为了解决匿名管道只能用于亲缘关系的进程间通信
   - 提供了一个路径名与之关联，存在于文件系统中，只要访问该路径，就能相互通信，同样是FIFO
3. 信号Signal
   - 信号可以在任何时候发给某一进程而无需知道该进程的状态
   - 如果该进程未执行，该信号会由内核保存起来，直到该进程恢复执行并传递给它
   - 信号是软件层次上对中断机制的一种模拟，是一种异步通信方式，主要来源于硬件和软件终止
4. 消息队列 Message
   - 存放在内核中的消息链表，每个队列由队列标识符表示
   - 存放在内核中，只有在内核重启或显示删除时才会被真正删除
   - 不一定是FIFO,可以按消息类型读取
5. 共享内存 Share Memory
   - 使得多个进程可以直接读写同一块内存空间，是最快的IPC方式
   - 内核专门留出一块内存区，需要访问的进程将其映射到自己的私有地址空间
   - 需要依靠某种同步机制来保证进程间的同步及互斥
6. 信号量 Semaphore
   - 计数器，用于多进程对共享数据的访问，主要用于进程间同步
7. 套接字 Socket
   - 让不再同一台计算机但通过网络连接计算机上的进程进行通信
   - 是支持TCP/IP的网络通信基本操作单元，是不同主机之间的进程双向通信的端点