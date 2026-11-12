# GameMsgType

packetId 14 的子类型。信封：`[type u8][levelSeq u8][sourcePlayer u8]`，其后
为各类型专属 payload（见 [gamemsg/](../gamemsg/)）。

`levelSeq` = 玩家当前 levelChangeCount（由 [LevelUpdatePacket](../LevelUpdatePacket.md)
驱动自增）。客户端分发器在 GameMsg 入口做 stale 闸门：`payload.levelSeq != 本地 lvSeq`
→ 整包丢弃（sub_1A99E98 的 `v27[1] == *a1` 比较）。**观众族请求（19/20/21/22/24）
在闸门之前分流，不受此检查约束**（handleGameMsg 先行分支）。

| type | 名称 | 方向 | payload | 验证 |
|-|-|-|-|-|
| 2 | NetRpc | ◆ | [NetRpc 记录流](../types/NetRpc.md) | [capture] |
| 3 | PlayerStateDelta | ◆ | [快照帧](../types/snapshot.md)：S→C `[size u32][playerId u8][state]×n`；C→S 单条 | [capture] |
| 4 | Critters | ◆ | 不透明转发（服务器不解析） | [capture] |
| 7 | Affinity | ▲ | `[compressed count ≤8][{netPlayerId u8, value u8}]×n`（value/255 归一化） | [capture] |
| 8 | NetLevelDataElect | ▼ | [快照帧] + [NetLevelData](../types/NetLevelData.md)（1047B 初始关卡数据） | [capture] |
| 9 | NetLevelDataRevoke | ▼ | 权威让位触发；非权威发送被忽略 | [asm] |
| 10 | NetLevelDataRevokeAck | ▲ | 带完整关卡数据 | [capture] |
| 11 | NetLevelData | ◆ | [快照帧] + [NetLevelData](../types/NetLevelData.md) | [capture] |
| 12 | NetLevelDataHeartbeat | ▲ | `[ready bool][playerId u8][levelId u32][authorityVersion u16]`；无权威时触发选举 | [capture] |
| 14 | SnapshotAck | ◆ | `[playerAckSeq u8][levelAckSeq u8]` | [capture] |
| 15 | MusicSync | ◆ | 不透明转发；观测结构 = `[轨道名\0][f64 进度]…` | [capture] |
| 16 | Metrics | ▲ | 客户端遥测，仅记录 | [impl] |
| 17 | NetLevelElectionNominee | ◆ | `[levelId u32][reason u16]`；官方服务器向同关其他玩家转发（322 上 / 74 下） | [capture] |
| 19 | AudienceHint | ▲ | 见 [audience/](../audience/)，不可靠通道 | [asm] sub_1A9B810 a5=0 |
| 20 | AudienceSocialBroadcast | ▲ | 见 audience/，不可靠 | [asm] a5=0 |
| 21 | AudienceConsensusVote | ▲ | 见 audience/，可靠 | [asm] a5=1 |
| 22 | AudienceChat | ▲ | 见 audience/，可靠 | [asm] a5=1 |
| 23 | LevelStateSideChannel | ▲ | `[levelId u32][ffffffff][0000][ffff]`，仅记录 | [capture] |
| 24 | AudienceSpotlightRequest | ▲ | 见 audience/，可靠 | [asm] a5=1 |

> 客户端分发器（sub_113AB40）调试名：19=`audience hint`、20=`audience social
> broadcast`、21=`audience consensus vote`、22=`audience chat`、
> 24=`audience spotlight req`——与上表一一对应。
