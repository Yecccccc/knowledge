### webrtc qos
1. nack & fec配合策略

  设计阈值，当rtt高于阈值，完全使用fec，低于阈值，完全使用nack
  
  low rtt：only ack
  
  high rtt：only fec
  
  medium rtt：混合模式hybrid mode(根据丢包率、bits/frame查表，决定帧的保护系数)
  
2. ulpfec && red封装

3. bbr状态机

  - startup阶段：每次使用2/ln2（1.143）的pacing gain增加发送速率，当连续3个rtt，交付速率不增加超过25%，bbr认为达到了瓶颈，进入drain阶段
  - drain阶段：使用startup阶段pacing_gain的倒数作为新的pacing_gain，快速排空startup阶段产生的队列，当inflight（网络管道中的数据包总字节数）与BDP（带宽时延积）匹配时，进入probebw阶段
  - probebw阶段：探测带宽，使用一个序列化的pacing_gain：1.25，0.75，1，1，1。每个阶段维持一个RTprop的长度
  - probertt阶段：RTprop在10s内未被更新，减少cwnd到BBRMinpipeCwnd（4个包），维持少量传输至少ProbeRTTDuration=200ms，探测得到RTT，再进入StartUp或者Probe BW阶段

4. bbrv1的缺点
   - 过于激进的startup阶段：启动探测容积大，形成深队列，影响其他tcp流，造成拥塞
   - 过于接近的带宽探测：存在多个BBRv1，会高估整个链路的容量，导致形成大量丢包
   - 不公平的RTT探测：RTT大的BBRv1流占用更多的带宽
   - 不公平竞争：BBRv1对丢包无感

5. rtmp协议
   - 应用层协议，基于tcp，延时1-2s，使用端口号为1935，视频封装格式为flv-tag，是连续的流
   - rtmp的每一个message就是一个视频帧，每一个flv-tag就是一个message
   - message format：header（length、timestamp、message type id、message stream id），message payload
   - 支持多路复用：可以将不同的视频流切片在同一连接传输
   - chunk：rtmp以message为基本数据单位，以chunk（message拆分为chunk）为传输单位，同一message的不同chunk有先后的串行发送，先发送的chunk一定先到达
   - 将message切分为chunk的目的：防止一个大的数据包传输时间过长，影响其他包
   - chunk format：header（basic header、message header、extended timestamp）、chunk data
   - 整体是协议先行、数据次之

6. rtmp优点/缺点/tips
   **优点**：
   - 低延迟（1-2s），rtmp使用独占的1935端口，无需缓冲
   - 适应性强：所有rtmp服务器都可以录制直播媒体流，同时允许观众跳过部分广播并且在直播开始后加入流
   - 灵活性：rtmp支持整合文本、视频、银牌
  **缺点**：
   - html5不支持
   - 容易收到带宽问题的影响：出现低带宽问题，造成视频中断
   - http不兼容
  **接收端如何判断哪些chunk属于同一个message**
   - 通过srs代码可知：接收端将接收到的chunk的chunk data相加，如果等于message payload则认为属于一个message
   - csid与chunk类型有关，同一类型不同chunk csid相同
   - msid用于标识哪些chunk属于同一个message，csid用于标识流通道

7. NAT(network address translation)
   ipv4网络可用ip数量为256^4，IP地址不够用，nat划分公网、子网，将一个公网ip下的子网ip发送的数据包转换成自身的数据包进行发送
   - 完全圆锥形：把一个来自内部ip地址和端口的所有请求都映射到相同的外网IP地址和端口（无限制，任何主机都可以通过外网ip端口向内部主机通信）
   - 地址受限形：与完全圆锥形不同的地方在于，当内网主机向某公网主机发送报文后，只有对应ip的公网主机才能向内网发送报文（ip受限，端口不受限）
   - 端口受限形：在ip受限的基础上，只有对应公网ip和端口才能向内网发送报文
   - 对称性nat：内部地址每次向不同外部主机发送数据时，会随机选择一个公网ip/port向外发送数据（私网ip/port与对端ip/port一一对应）

8. ice, stun, turn, sdp
   stun：需要一个公网的server，实现一个基于client-server的协议，由客户端主动发起连接，stun server会通知客户端它的公网IP以及端口
   **对称性nat无法使用，因为对称式nat中私网ip/port与对端ip/port一一对应**
   turn：使用中继穿透nat，针对对称性nat
   **ice服务器**
   - 搜集地址：通话前收集本地ip和端口号（candidate），然后用candidates与stun server通信，得到可用的candidate与公网ip与端口号。然后再访问turn得到所有的中继candidate以及公网ip/port
   - 检查连接：主叫和被叫交换sdp，拿到对方的candidates后，使用这些地址发送stun消息，检查连接
   - 完成
  **sdp**：包含了一些描述媒体会话信息的文本，可以用于在两个对等的端之间协商媒体会话的参数，例如：编解码器、媒体流的传输方式。包含：会话版本号、会话名称、会话描述协议、连接信息、媒体描述信息、流格式等。
