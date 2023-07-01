Interview preparation for [About Me](about/README.md)

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
Bevy, Axum