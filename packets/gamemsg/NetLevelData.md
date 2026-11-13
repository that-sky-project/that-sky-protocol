# GameMsgType_NetLevelData 族 (8 / 9 / 10 / 11) ◆

关卡权威数据通道。8=Elect（当选下发）、9=Revoke（让位）、10=RevokeAck、
11=普通同步。

> **转发边界**：本包正常情况**不解字段，直接转发**——服务器必须介入的
> 仅两处：快照层（对每个接收者维护独立差分窗口），以及 Elect(8)/Revoke(9)
> 的权威生命周期事件（见 [../flows/authority.md](../flows/authority.md)）。
> 明文结构的字段级解析只在存储初始关卡数据（LevelStateStore）时需要，
> 且只取 electedPlayer / levelId / authorityVersion / dataLength / levelData。

**信封后是 [快照帧](../types/snapshot.md)**，差分解出的明文
才是 NetLevelData 结构。

## NetLevelData 明文布局（IDA `sub_1A80EA8` 逐字节证实）

| 偏移 | 字段 | 类型 | 说明 |
|-|-|-|-|
| 0 | electedPlayer | u8 | 包头玩家 id（真实权威由会话状态确定，此字段可为 0，不作授权依据） |
| 1 | **levelId** | u32 LE | **在偏移 1**（sub_1A80EA8 序列化顺序铁证） |
| 5 | **authorityVersion** | u16 LE | 选举纪元（见下） |
| 7 | hasData | **1 bit** + 字节对齐 | 有无关卡数据 |
| 8 | unknown2 | u16 LE | |
| 10 | unknown3 | u8 | |
| 11 | levelHash | u32 LE | netVersion 首字 |
| 15 | mergeState | u8（compressed 0..255） | NetLevelMergeState |
| 16 | unknown5 | u16 LE | |
| 18 | unknown6 | u16 LE | |
| 20 | dataLength | u16 LE | ≤ 0x2000 |
| 22 | levelData | u8[dataLength] | 客户端序列化的关卡流 |

空包体总长 **22 字节**（官方 Elect 黄金向量）。

> 历史勘误：早期从抓包误读为 `[electedPlayer u8][unknown0 u8][unknown1 u16]
> [hasData bit][levelId u32@偏移5]`，是旧实现的错位根源（levelId 解成
> 0x01000101、重编码清零字节 8），已按原生布局修正。

## authorityVersion 生命周期

它计数**选举**，不是每次 LevelUpdate：

- 首次 Elect version=2；当选客户端 C→S type11 上传仍为 2；
- S→C type11 投影为 3；后续真实重选 Elect=4（+2 步进，u16 回绕）。
- 服务器据此拒绝过期上传；同代际重复 LevelUpdate 幂等，不重发 Elect。
- 官方 Elect 只在客户端 LevelUpdate 之后下发；JoinGame 阶段不发。

## 流程

- **Elect (8)**：关卡当选时服务器（或客户端解出）向其他玩家下发，payload
  含初始关卡数据（官方抓包 1047B）。客户端把它当"权威已定"。
- **Revoke (9)**：权威让位。**只有权威本人可发**——非权威发送被忽略；
  权威发送会触发惩罚 + 重选（见 [flows/authority.md](../flows/authority.md)）。
- **RevokeAck (10)**：让位确认，带完整关卡数据（763B）。
- **同步 (11)**：权威持续上报 → 服务器存 LevelStateStore → 每 tick 向
  非权威玩家经 levelDelta SnapshotWriter 差分下发。

验证：**[capture]**（1831B 双向样本、1047B Elect、763B RevokeAck）+
**[asm]**（sub_1A80EA8 原语序列）+
**[golden]**（`tools/verify-level-election-capture.mjs`：50+270 Elect、
83075 条 type11 帧由现编解码 byte-exact 复算）。
