# GameMsgType_PlayerStateDelta (3) ◆

玩家状态的快照差分流。

> **转发边界**：本包正常情况**不解字段，直接转发**——服务器唯一必须
> 介入的是快照层（每个接收者独立差分窗口）。若需要位置数据（观测、
> 迁移），PlayerState 前 13 字节稳定可读即可，无需完整解布局。

信封后是 [快照帧](../types/snapshot.md)
`[save_seq][base_seq][fnv1a8 checksum][payload]`。

## C→S（客户端上报）

payload = **明文 PlayerState**（无条目包装）。服务器用自己的 SnapshotReader
差分解帧，得到明文记录（布局见 [types/PlayerState.md](../types/PlayerState.md)）。

## S→C（服务器广播）

payload = 条目流，读至包尾：

| 字段 | 类型 |
|-|-|
| size | u32 LE |
| playerId | u8（netId） |
| state | u8[size]（明文 PlayerState） |

服务器把每个房内玩家的最新状态编码为条目，按 0x2100 分批，**对每个
接收者用自己的 SnapshotWriter 差分**后发出。不足 2 名玩家跳过。

**stale 闸门**：信封 `levelSeq != 本地 lvSeq` → 丢弃（换图瞬间的旧状态
不得串进新关卡）。

解码失败兜底：PlayerState 前 13 字节在所有已知布局稳定
（`stateKind = b[0]&7`、`position = 3×f32@1,5,9`）——位置照常上报不踢人；
同一失败特征 30s 限频告警。

验证：**[capture]**（35453 下 / 61273 上，快照帧头逐字节）+ **[asm]** +
**[impl]**。
