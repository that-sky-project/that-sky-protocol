# GameMsgType_NetLevelDataHeartbeat (12) ▲

| 字段 | 类型 | 说明 |
|-|-|-|
| ready | bool | |
| playerId | u8 | |
| levelId | u32 LE | |
| authorityVersion | u16 | |

关卡权威保活（官方 39666 次）。服务器发现该关无权威时触发选举
（见 [flows/authority.md](../flows/authority.md)）。

验证：**[capture]** `0c 01 00 | 00 00 de872cd2 0000`。
