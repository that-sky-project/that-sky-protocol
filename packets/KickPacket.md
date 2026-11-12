# KickPacket (2) ▼

| 字段 | 类型 | 说明 |
|-|-|-|-|
| length | u32 LE | |
| reason | utf8 ≤ 256B | |

服务器强制踢出。不消耗 C→S 序号。
