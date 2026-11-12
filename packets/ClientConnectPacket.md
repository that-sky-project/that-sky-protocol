# ClientConnectPacket (0) ▲（占位）

ENet connect 事件本身即连接建立；此 id 是客户端本地构造的占位，**不产生
网络包**。服务器的 ENet `peer:connect` 处理即对应此阶段。
