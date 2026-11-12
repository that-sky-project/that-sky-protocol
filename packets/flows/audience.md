# 流程：千人房观众数据

```
入座：
房间管理器 ── seat.assign [playerId, benchId, seatId, entityId, roomMembers] ──▶ 房间服务器
房间服务器：占座（座位实体置 IN_ROOM）(flags=IN_ROOM) → FeatureFlags(32) (true,true)×2
      → RoomIds(22) [成员实体号, tail]
每帧（≥1s）：
  房间服务器 → RoomIds(22)、StreamInfo(28)、ReliableData(23) chunks、
         Audience(18) 剪影帧 ×n（630B 预算分页）
  房间服务器 ── seat.occupied 遥测事件（位置变化>0.25）──▶ 房间管理器
跨房同步：
  房间管理器 每秒快照轮 → audience.snapshot 命令
    [levelId, seats[]（他房座位，flags=OUTSIDE_ROOM）, removed[], full?, spotlightSession]
  房间服务器把幽灵座位写入 座位实体表 → 下一帧 id 18 带出
演出：
  客户端 ── SpotlightRequest(24) ──▶ 房间服务器 → SpotlightResponse(29) 两 bool
       ── 聚光灯变更事件 ──▶ 房间管理器 → 快照轮同步全厅
  演奏者 ── SocialBroadcast(20) ──▶ 房间服务器中继 SocialBroadcastDown(24)
       [sessionId=hash(演奏者), json]
  StreamInfo(28) sessionId 命中者 → 演奏状态（stage 索引 48 + startTime 锚点）
聊天：
  客户端 ── Chat(22) ──▶ 房间服务器路由（同排/控制消息）──▶ ChatDown(26/27)
```

关键不变量：
- entityId = bench×8 + seat + 1，≤10143；10144/10145 = 聊天控制 ID
- 档案 f0 = identityHash(playerId)（绑定链路的根）
- commitRecord=true 的剪影必须已有档案（先 23 后 18）
- 跨房幽灵座位的时间戳必须钳制到本房间服务器时钟域（源房间服务器超前的未来戳
  会让该侧剪影永不渲染——单向可见 bug 的根因）
