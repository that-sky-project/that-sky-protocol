# 流程：入房（Join）

```
客户端                                服务器
  │ ENet connect                        │
  │──── JoinGame(3) [uuid, levelId, lvSeq, move?] ──▶│
  │                          迁移？→ claimMigrationRoster（预占 netId）
  │                          顶号？→ evict 旧会话（PlayerLeft + seat.released）
  │                          房满/冲突？→ StateError 拒绝
  │◀─── EnterGame(17) 单发本人 ──────────│  ← 全房名单
  │◀─── EnterGame(17) 广播他人 ──────────│  ← 老玩家感知新人
  │     （权威：关卡无权威 → setAuthority('join_first_player')；
  │       有权威 → 给新人单独发 Elect）   │
  │◀─── PlayerStateDelta(14/3) 每帧 ─────│  ← 10Hz 快照差分
  │──── SnapshotAck(14/14) ────────────▶│
```

关键规则：
- 会话状态机 CONNECTED→PENDING→ACTIVE（→MOVING→CLOSED），非法转移抛错。
- **connMagic**：JoinGame 后续包 token 必须一致，否则坏包计数（8 次断开）。
- 一个 uuid 只允许一个活跃 Player；ENet peer = 一个 ConnectionSession。
