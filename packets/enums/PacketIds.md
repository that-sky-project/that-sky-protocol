# PacketIds

会话层包 id。方向：▲ = C→S，▼ = S→C，◆ = 双向。

| id | 名称 | 方向 | 说明 | 验证 |
|-|-|-|-|-|
| 0 | ClientConnect | ▲ | ENet 连接建立的占位（客户端本地构造，不发上服务器） | [asm] |
| 1 | Disconnect | ◆ | `[reason u8]`。客户端/服务器任意一侧主动断开 | [capture] |
| 2 | Kick | ▼ | `[len u32][utf8 reason ≤256B]` | [asm] |
| 3 | JoinGame | ▲ | 入房握手，见 [JoinGamePacket](../JoinGamePacket.md) | [capture] |
| 4 | LevelUpdate | ▲ | 换图上报，见 [LevelUpdatePacket](../LevelUpdatePacket.md) | [capture] |
| 5 | **SnapshotResync** | ▼ | 零载荷快照重置命令：客户端无条件重置两路快照 Writer/Reader 并写 0x0101 keyframe 标志（state+0x2E3C0）。旧表标 DEPRECATED 的 EnterGame 槽位，0.35.1 复活为此命令。服务器在会话快照窗口 `resyncfailed`（两次对半收缩无 ack）时下发 | [asm] sub_1A99E98 id-5 分支 |
| 6 | PlayerJoined | ▼ | 占位（0.35.1 走 EnterGame） | [asm] |
| 7 | PlayerLeft | ▼ | `[playerId u8][reason u8]` | [capture] |
| 8 | PlayerChangedLevel | ▼ | `[playerId u8][levelId u32 LE]` | [capture] |
| 9 | MoveGame | ▼ | 跨房迁移指示，见 [MoveGamePacket](../MoveGamePacket.md) | [capture] |
| 10 | MoveResult | ▲ | `[result u8][原样尾]`；result==0 = 客户端接受迁移 | [capture] |
| 11 | CancelMove | ◆ | `[reason u8]`；服务器在组队迁移失败时下发 reason 2 回滚 | [capture] |
| 12 | NetTimePing | ▲ | `[requestId u16]`，周期心跳 | [capture] |
| 13 | NetTimePong | ▼ | `[requestId u16][serverTime f64][reserved u32=0][scalar f32≈1.3–1.56]` | [capture] |
| 14 | GameMsg | ◆ | 大类容器，22 个子类型见 [GameMsgType](GameMsgType.md) | [capture] |
| 17 | EnterGame | ▼ | 房间名单推送，见 [EnterGamePacket](../EnterGamePacket.md) | [capture] |
| 18–33 | Audience 族 | ▼ | 千人房观众系统，见 [audience/](../audience/) | [capture] |
| 34 | AudienceDebugBlob | ▼ | `[kNetMsg_AudienceDebugBlob]` 纯调试，客户端无 handler | [asm] |

**确认不存在/不实现**：
- id 21（S→C 9 字节 `[u32][u8][u32]`）：客户端解析后**直接丢弃**（sub_1A7F968
  的调用点无任何消费者）——死槽位。
- id 128（▲，1 字节 `06`）：官方客户端发送原语无此调用——mod 注入产物，
  抓包 5 次，全部来自带 mod 的客户端。
