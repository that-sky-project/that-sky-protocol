# GameMsgType_NetLevelData 族 (8 / 9 / 10 / 11) ◆

关卡权威数据通道。8=Elect（当选下发）、9=Revoke（让位）、10=RevokeAck、
11=普通同步。

> **转发边界**：本包正常情况**不解字段，直接转发**——服务器必须介入的
> 仅两处：快照层（对每个接收者维护独立差分窗口），以及 Elect(8)/Revoke(9)
> 的权威生命周期事件（见 [../flows/authority.md](../flows/authority.md)）。
> 明文结构的字段级解析只在存储初始关卡数据（LevelStateStore）时需要，
> 且只取 electedPlayer / levelId / dataLength / levelData 四个字段。

**信封后是 [快照帧](../types/snapshot.md)**，差分解出的明文
才是 NetLevelData 结构。

## NetLevelData 明文布局

| 偏移 | 字段 | 类型 | 说明 |
|-|-|-|-|
| 0 | electedPlayer | u8 | 权威玩家 netId |
| 1 | unknown0 | u8 | 官方样本值 1 |
| 2 | unknown1 | u16 LE | 官方样本值 74 |
| 4 | hasData | **1 bit** + 字节对齐 | 有无关卡数据 |
| 5 | **levelId** | u32 LE | **在偏移 5**（bit-probe 铁证；旧实现放偏移 1，每个官方包解出 0x01000101 且重编码清零字节 8——新玩家初始世界状态的关卡 id 损坏，已修） |
| 9 | unknown2 | u16 LE | |
| 11 | unknown3 | u8 | |
| 12 | levelHash | u32 LE | |
| 16 | mergeState | u8（compressed 0..255） | NetLevelMergeState |
| 17 | unknown5 | u16 LE | |
| 19 | unknown6 | u16 LE | |
| 21 | dataLength | u16 LE | ≤ 0x2000 |
| 23 | levelData | u8[dataLength] | 客户端序列化的关卡流（暗影/新地图生成等） |

## 流程

- **Elect (8)**：关卡当选时服务器（或客户端解出）向其他玩家下发，payload
  含初始关卡数据（官方抓包 1047B）。客户端把它当"权威已定"。
- **Revoke (9)**：权威让位。**只有权威本人可发**——非权威发送被忽略；
  权威发送会触发惩罚 + 重选（见 [flows/authority.md](../flows/authority.md)）。
- **RevokeAck (10)**：让位确认，带完整关卡数据（763B）。
- **同步 (11)**：权威持续上报 → 服务器存 LevelStateStore → 每 tick 向
  非权威玩家经 levelDelta SnapshotWriter 差分下发。

验证：**[capture]**（1831B 双向样本、1047B Elect、763B RevokeAck）+
**[asm]**（sub_1A80EA8 原语序列 + bit-probe levelId@偏移5）。
