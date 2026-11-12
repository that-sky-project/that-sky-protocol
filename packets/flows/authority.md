# 流程：关卡权威选举

关卡数据由**客户端权威**持有（第一个进关的玩家）。完整时序（官方抓包
999 事件还原）：

```
1. UP   NetLevelDataHeartbeat(12) ×N        ← 玩家持续保活
2. 服务器发现该关无权威
3. UP   NetLevelElectionNominee(17)          ← 玩家提名自己
4. 服务器 DOWN Nominee(17) 转发给同关他人     ← 322 上/74 下（选择性）
5. 服务器 DOWN NetLevelDataElect(8)          ← 1047B 初始关卡数据
6. UP   NetLevelDataHeartbeat 继续（新权威）
7. （旧权威让位）UP RevokeAck(10) 763B       ← 关卡数据完整移交
```

惩罚：权威发 Revoke(9) → 惩罚名单 + 全关 Revoke 广播 + 重选；非权威
发 Revoke 直接忽略。只剩一人且受罚 → 豁免。

XYSKY UDP 对应：对应实现：权威再平衡、Elect 广播抑制、当选下发。
