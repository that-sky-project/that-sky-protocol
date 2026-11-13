# 流程：关卡权威选举

关卡数据由**客户端权威**持有（第一个进关的玩家）。完整时序（官方抓包
999 事件还原）：

```
1. UP   JoinGame(lvSeq=0)                    ← 不发 Elect（铁证：官方抓包）
2. UP   LevelUpdate(lvSeq=1)                 ← 客户端代际就绪
3. UP   NetLevelDataHeartbeat(12) ×N        ← 玩家持续保活
4. 服务器发现该关无权威
5. UP   NetLevelElectionNominee(17)          ← 玩家提名自己
6. 服务器 DOWN Nominee(17) 转发给同关他人     ← 322 上/74 下（选择性）
7. 服务器 DOWN NetLevelDataElect(8)          ← 只发给已 LevelUpdate 的玩家；
                                               首次 version=2，空包体 22B
8. 当选客户端 UP NetLevelData(11) version=2（初始关卡数据上传）
9. 服务器 DOWN NetLevelData(11) version=3   ← 向其他已就绪玩家投影
10. UP   NetLevelDataHeartbeat 继续（新权威）
11. （旧权威让位）UP RevokeAck(10) 763B       ← 关卡数据完整移交
12. 真实重选 → Elect version=4（每次选举 +2，u16 回绕）
```

同代际重复 LevelUpdate 幂等（不重发 Elect）；lvSeq 变化或换房主才向
该玩家重发当前选举。

惩罚：权威发 Revoke(9) → 惩罚名单 + 全关 Revoke 广播 + 重选；非权威
发 Revoke 直接忽略。只剩一人且受罚 → 豁免。

XYSKY UDP 对应：对应实现：权威再平衡、Elect 广播抑制、当选下发。
