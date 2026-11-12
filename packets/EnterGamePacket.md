# EnterGamePacket (17) ▼

房间名单推送。新玩家 JoinGame 完成后**先单发给本人**，再广播给房内其他人
（老玩家靠它感知新人）。也是迁移组"全员到齐"名单的载体。

| 字段 | 类型 | 说明 |
|-|-|-|-|
| playerCount | compressed(0,8) | ≤ 8（房间硬上限） |
| playerList | 每人： | |
| — netId | u8 | 服务器分配的房间内槽位（1..0xFF） |
| — playerUuid | [TgcUuid](types/TgcUuid.md) | 16 字节身份 |
| — reserved | u8[16] | 全零填充 |
| — levelId | u32 LE | 该玩家当前关卡 |

验证：**[capture]** 官方样本 `01 8310ba6d…`（count=1 + 48B 条目）；
**[impl]** 迁移组到齐名单。
