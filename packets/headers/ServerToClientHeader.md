# ServerToClientHeader

每个 S→C ENet payload 以 3 字节定长头开始。

| 偏移 | 字段 | 类型 | 说明 |
|-|-|-|-|
| 0 | packetId | u8 | [PacketIds](../enums/PacketIds.md) |
| 1 | payloadLength | u16 LE | 后随 payload 字节数（>0xFFFF 报错） |