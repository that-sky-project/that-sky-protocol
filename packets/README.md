# Sky: CotL UDP Game Protocol — 0.35.1

ENet 可靠 UDP 游戏会话协议（本目录）。HTTP(S) REST 请求见 [../requests/](../requests/)

---

## 1. 分层模型

```
┌────────────────────────────────────────────────────────────┐
│ ENet 可靠 UDP（checksum 开启，应用层只用 channel 0）          │
├────────────────────────────────────────────────────────────┤
│ 会话头   C→S: [token u32][packetId u8][sequence u8]         │
│          S→C: [packetId u8][payloadLength u16 LE]           │
├────────────────────────────────────────────────────────────┤
│ 包层     按 packetId 分发 → 本目录 *.md                      │
├────────────────────────────────────────────────────────────┤
│ GameMsg  packetId 14 内嵌 [type][levelSeq][sourcePlayer]     │
│          → 22 个子类型 → gamemsg/                           │
├────────────────────────────────────────────────────────────┤
│ 快照层   PlayerState / NetLevelData 走 XOR+RLE 差分窗口      │
│          → types/snapshot.md                                │
└────────────────────────────────────────────────────────────┘
```

## 2. 会话生命周期

```
客户端                              服务器
  │ ENet connect                      │
  │──── JoinGame (3) ────────────────▶│  ← 会话建立，connMagic = token
  │◀─── EnterGame (17) ───────────────│  ← 全房玩家列表
  │◀─── 状态推送（每 tick）───────────│
  │     GameMsg 3/11/14（快照差分）    │
  │──── GameMsg 14/14 SnapshotAck ───▶│
  │──── LevelUpdate (4) ─────────────▶│  ← 每次换图
  │──── NetTimePing (12) 周期 ───────▶│
  │◀─── NetTimePong (13) ─────────────│
  │     （换房迁移见 flows/room-move） │
  │──── Disconnect (1) / ENet 断开 ──▶│
```

权威模型：**服务器**权威于房间名单、迁移决策、座位实体分配；
**客户端**权威于自己的 PlayerState 流与所在关卡的 LevelData（第一个进入
关卡的玩家经 NetLevelDataElect 当选 authority）。详见 [flows/](flows/)。

## 3. 转发与解析边界（实现者须知）

- **PlayerState（GameMsg 3）与 NetLevelData（GameMsg 8/11）正常情况
  不需要解字段，可直接转发**——服务器唯一必须介入的是**快照层**：对每个
  接收者维护独立差分窗口（Writer），并在转发前用对应 Reader 解帧。
  字段级解析只在需要位置数据（遥测/迁移）时对 PlayerState 做，且前
  13 字节（stateKind + position）在所有已知布局中稳定，够用。
- NetRpc / Critters / MusicSync 同理整体透传；NetRpc 的例外是 gift
  accept 的 receiver uuid 改写（防伪造），见 gamemsg/NetRpc.md。
- 解码失败兜底：PlayerState 前 13 字节在所有已知布局稳定
  （stateKind=b[0]&7、position=3×f32@1），位置照常上报不踢人。

## 4. 目录

| 目录 | 内容 |
|---|---|
| [headers/](headers/) | C→S 与 S→C 会话头 |
| [enums/](enums/) | PacketIds 全表、GameMsgType 全表、常量 |
| packets/*.md（本目录） | 会话层包（id 0–17） |
| [gamemsg/](gamemsg/) | packetId 14 的 22 个子类型 |
| [audience/](audience/) | 千人房观众系统包（id 18–33 + 上行 GameMsg 族） |
| [types/](types/) | 复用结构（PlayerState、快照、位原语、身份哈希…） |
| [flows/](flows/) | 端到端流程：join / level / authority / room-move / audience |

## 5. 约定

- 多字节整数一律 **小端**（LE）。
- 位流原语见 [types/bit-primitives.md](types/bit-primitives.md)。
- 每个文档标注验证状态：**[capture]** = 官方抓包逐字节复算；
  **[asm]** = 客户端汇编；**[impl]** = 与 XYSKY UDP 实现互通验证；
  未标注 = 结构性推断。