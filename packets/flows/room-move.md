# 流程：跨房迁移（串线/合并）

**ENet 连接不转移**——客户端驱动重连。协调全部经 房间管理器 管理器。

```
房间管理器 ── move.prepare 命令 [moveId, playerIds, expectedVersion] ──▶ 目标房间服务器
      （目标按 telemetrySeq 做 STALE_PLAN 检测 + 容量预留，TTL 15s）
房间管理器 ── move.commit 命令 [playerIds, toRoomId, udpHost, udpPort] ──▶ 源房间服务器
源房间服务器：
  movePlayersAtomic：≤7 人全部 ACTIVE → 每人发 MoveGame(9)
    [address=目标, players=全组, moveNonce, isSplit=true, destinationUuid]
  失败回滚 → CancelMove(11) reason=2
客户端：
  收 MoveGame → MoveResult(10) result=0 → 断开 ENet
  → 连目标 → JoinGame(moving=true, move.players=全组)
目标房间服务器：
  迁移名册认领（预占 netId）：匹配预留 → 为每人预分配 netId → 15s 内全组到齐
    → migration complete → 发 move.completed 事件
  未到齐 → 预占 netId 释放
源房间服务器：
  收 MoveResult → 房间管理器 move.result 事件；20s 卡住 → 强制断开
```

失败闭环：房间管理器 侧 prepared 15s / committed 30s 超时 → failed +
move.cancel 重投队列；客户端拒绝（result≠0）→ 部分失败 cancel 剩余。
