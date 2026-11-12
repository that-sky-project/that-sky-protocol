# Constants

| 名称 | 值 | 说明 |
|-|-|-|
| kNetRpc_MaxTypes | 256 | NetRpc 类哈希数量上限（wire u16 bodyLen bounded 0x1000） |
| kAudienceMaxEntityIndex | 10143 | 座位实体号上限（bench×8+seat+1）；聊天控制 ID 10144/10145 在其上 |
| kAudienceEntityLimit | 10171 | 客户端 entityLimit 浮点钳位（0x27DB） |
| SNAPSHOT_MAX_DATA_BYTES | 0x2100 | 快照解码缓冲上限 |
| NET_LEVEL_DATA_MAX_BYTES | 0x2000 | NetLevelData 数据区上限 |
| PLAYER_STATE_MAX_BYTES | 0x4B5 | PlayerState 记录上限 |
| snapshot 窗口 | 16 槽 | keyframe 间隔默认 10 |
| 身份哈希 | FNV-1a-64 | offset 0xCBF29CE484222325 / prime 0x100000001B3，见 types/identity-hash.md |
