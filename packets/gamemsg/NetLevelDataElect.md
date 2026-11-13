# GameMsgType_NetLevelDataElect (8) ▼

见 [NetLevelData.md](NetLevelData.md)。当某关第一个玩家出现（或旧权威
离开后 rebalance），服务器/客户端向新权威下发 Elect，payload = 快照帧 +
NetLevelData（含初始关卡数据）。官方抓包 270 次；**只在客户端
LevelUpdate 之后下发**（JoinGame 阶段不发），首 Election version=2，
空包体 22 字节。
