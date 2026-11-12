# JoinGamePacket (3) ▲

入房握手。ENet connect 后的第一包，服务器以此绑定会话（connMagic = 会话头
token）。字段序经官方抓包复算与房间服务器互通双验证。

| 偏移 | 字段 | 类型 | 说明 |
|-|-|-|-|
| 0 | uuid | [TgcUuid](types/TgcUuid.md) | 玩家身份。|
| 16 | sessionUuid | TgcUuid | 会话 uuid |
| 32 | sessionToken | u8[32] | 会话令牌（服务器解出但不消费） |
| 64 | move | [Move](types/Move.md) | 组队迁移块（见下） |
| … | joinToken | u32 | |
| … | joinState | u8 | |
| … | joinFlags | u32 | |
| … | joinSignature | u8[16] | |
| … | serverVersion | u16 | |
| … | clientVersion | u16 | |
| … | rpcHash | u32 | |
| … | netVersion | [NetVersion](types/NetVersion.md) | 6×u16，首项 = levelHash |
| … | levelChangeCount | u8 | 初始 lvSeq（GameMsg stale 闸门基准） |
| … | levelId | u32 | 当前关卡（FNV-1a-32 of 关卡名） |
| … | levelSummary | u8[64] | |
| … | localPlayerId | u32 | |
| … | currentPlayerId | u32 | |
| … | levelDataLength | u32 | ≤ 0x2000 |
| … | levelData | u8[len] | 携带关卡数据入房 |
| … | moving | bool | true = 这是迁移重连 |
| … | hasTransferAddress | bool | |
| … | mergeState | compressed u8(0,255) | |
| … | transferAddress | typed [NetAddress](types/NetAddress.md) | |
| … | moveResult | u32 | |

**服务器实际消费**：`uuid`、`levelId`、`netVersion`、`levelChangeCount`、
`move`（迁移认领）、`moving`。其余解出保留。

## Move 块与迁移重连

`moving=true` 时服务器的处理见 [flows/room-move.md](flows/room-move.md)：
迁移组通过 `move.players`（≤7 个 uuid）在目标服务器上预占 netId（migrationRoster），
全组到齐后统一完成。JoinGame 的 `move.players` 必须包含自己的 uuid。
