# Move

迁移块（JoinGame 内嵌 / MoveGame 主体）。

| 字段 | 类型 | 说明 |
|-|-|-|
| address | [NetAddress](NetAddress.md) | 目标 IPv4:port（6B） |
| moveNonce | u64 | 关联 id |
| splitType | u8 | |
| playerCount | u32 | ≤ 7 |
| players | [TgcUuid] ×count | 组员名单（必须含本人） |
