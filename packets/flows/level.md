# 流程：换图（LevelUpdate）

```
客户端 ─── LevelUpdate(4) [lvSeq, levelId] ───▶ 服务器
   player.changeLevel()：lvSeq 更新
   旧关权威？→ revokeAuthority('authority_left_level') + rebalance
   新关无权威？→ 选举（第一个活跃未惩罚玩家）
   新关有权威？→ 给该玩家单独发 Elect（带初始关卡数据）
   广播 PlayerChangedLevel(8)
```

**stale 闸门**：换图瞬间旧关的 GameMsg（levelSeq 不匹配新 lvSeq）被丢弃，
防止旧关卡消息串进新关卡。观众族请求（19/20/21/22/24）例外。

权威数据同步（每 tick）：
- 权威玩家 ── NetLevelData(14/11 上行) ──▶ 服务器存 LevelStateStore
- 服务器 ── levelDelta 快照差分 ──▶ 非权威玩家
- 非权威 ── SnapshotAck ──▶ 裁剪窗口
