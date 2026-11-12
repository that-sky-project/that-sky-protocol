# MoveGamePacket (9) ▼

跨房迁移指示。服务器收到管理器 `move.commit 命令` 后向源房玩家下发；客户端
收包后自行断开 ENet、连接 `move.address`、以 `moving=true` 的 JoinGame
重新入房。**ENet 连接不转移**——见 [flows/room-move.md](flows/room-move.md)。

| 字段 | 类型 | 说明 |
|-|-|-|-|
| rawBytes32 | u8[32] | 不透明前缀（实现按全零填充） |
| address | [NetAddress](types/NetAddress.md) | 目标服务器 IPv4:port |
| moveNonce | u64 | 迁移关联 id；服务器生成 = `(Date.now()<<16) ^ (playerId<<8) ^ roomIdLen` |
| splitType | u8 | 0 |
| players | u32 count + [TgcUuid]×n (≤7) | 迁移组全名单 |
| u32Field | u32 | |
| u08Field | u8 | |
| u32Field2 | u32 | 默认 0xFFFFFFFF |
| isSplit | bool | true = 组队分片迁移 |
| destinationUuid | [TgcUuid](types/TgcUuid.md) | 目标房间身份 = `00000000-0000-4000-8000-<roomId 12 hex>` |

> 盲区：`rawBytes32`、`u32Field/u08Field/u32Field2` 按默认值填充。若真实
> 客户端校验这些字段，迁移握手会静默失败——目前实测通过。

验证：**[impl]**（迁移全链路互通）+ **[asm]**（字段序与 MoveGamePacket 结构一致）。
