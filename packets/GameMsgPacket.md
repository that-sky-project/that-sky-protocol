# GameMsgPacket (14) ◆

大类容器。信封后按 type 分发——22 个子类型的完整定义见 [gamemsg/](gamemsg/)
与 [audience/](../audience/)。

## 信封

| 字段 | 类型 | 说明 |
|-|-|-|-|
| gameMsgType | u8 | [GameMsgType](enums/GameMsgType.md) |
| levelSeq | u8 | 玩家 lvSeq；**stale 闸门**：与接收方本地 lvSeq 不符 → 整包丢弃（观众族请求 19/20/21/22/24 例外，先行分流不受闸门约束） |
| sourcePlayer | u8 | 原始发送者 netId（C→S 时由服务器改写为转发者 id） |

## 子类型索引

| 分组 | type | 文档 |
|-|-|-|
| 玩家状态 | 3 | [gamemsg/PlayerStateDelta.md](gamemsg/PlayerStateDelta.md) |
| 玩家状态确认 | 14 | [gamemsg/SnapshotAck.md](gamemsg/SnapshotAck.md) |
| 关卡数据 | 8 / 9 / 10 / 11 / 12 / 17 / 23 | [gamemsg/](gamemsg/) |
| RPC | 2 | [gamemsg/NetRpc.md](gamemsg/NetRpc.md) |
| 透明转发 | 4 / 15 | [gamemsg/Critters.md](gamemsg/Critters.md) / [MusicSync.md](gamemsg/MusicSync.md) |
| 社交 | 7 / 16 | [gamemsg/Affinity.md](gamemsg/Affinity.md) / Metrics |
| 观众上行 | 19 / 20 / 21 / 22 / 24 | [audience/](../audience/) |
