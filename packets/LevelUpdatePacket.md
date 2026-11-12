# LevelUpdatePacket (4) ▲

玩家换图上报。驱动服务器的 `player.changeLevel()`（lvSeq 自增）、权威
重选、以及 S→C `PlayerChangedLevel (8)` 广播。

| 字段 | 类型 | 说明 |
|-|-|-|-|
| levelChangeCount | u8 | 玩家 lvSeq（服务器采用或 +1） |
| netVersion | [NetVersion](types/NetVersion.md) | 6×u16，首项 = levelHash |
| levelStatus | u8 | |
| levelId | u32 LE | FNV-1a-32(关卡名) |
| levelSummary | u8[64] | |
| trailingState | bool | |

验证：**[capture]**（官方 LevelUpdate 流量存在且与房间服务器互通）。
