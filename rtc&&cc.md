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
