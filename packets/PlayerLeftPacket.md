# PlayerLeftPacket (7) ▼

| 字段 | 类型 | 说明 |
|-|-|-|-|
| playerId | u8 | 离开玩家的 netId |
| reason | u8 | 0 = 正常断开（抓包值） |

只发给 ACTIVE 会话（`playerOnly` 广播）。顶号（同 uuid 二次进房）时也发。

验证：**[capture]** 官方样本 `57 01` / `58 01` / `50 01`（id=0x57/0x58/0x50, reason=1）。
