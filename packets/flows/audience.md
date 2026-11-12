# 流程：千人房观众与演出

## 入座与观众同步

```text
客户端进入支持观众系统的关卡
  → 服务器分配 bench / seat / entityId
  → FeatureFlags(32) 启用观众系统
  → RoomIds(22) 绑定本房成员实体
  → ReliableData(23) 建立档案和身份哈希
  → Audience(18) 持续更新剪影位置
```

跨房实现需要汇集同关卡其他房间的座位、档案和位置，并向各房间分发。
每个接收房排除由普通房内玩家同步承载的成员，避免同时绘制完整玩家和剪影。
管理器与房间服务器的内部通信形式不属于客户端 UDP 线协议。

## 申请、排队与上台

```text
客户端 → SpotlightRequest（上行 GameMsg 子类型 24）
  → 服务器检查当前会话和申请者资格
  → 空闲：建立当前演出；忙碌：进入等待队列
  → SpotlightResponse(29) 确认申请已接受或被拒绝
  → StreamInfo(28) 按接收者提供：
       当前演奏者 sessionId 身份哈希
       接收者自身排队索引（旧字段名 stageEnd）
       本场广播截止时间（旧字段名 startTime）
```

接受申请与授予演奏权是两个步骤。排队者收到申请成功回执后，仍旁观当前
演出。客户端用 `sessionId == FNV-1a-64(自己的 UUID 原始字节)` 判定自己
是否为当前演奏者，不靠回执决定上台。

排队索引按有符号 i16 使用：-1 为未排队，0 表示下一位，1 表示第 2 位。
同一场演出向不同接收者提供的排队索引可以不同。

## 演出限时

每场由服务器确定截止时刻。客户端查询剩余时间的公式为：

```text
remaining = max(parsedDeadline - clientClock, 0)
```

`StreamInfo.startTime` 旧名实际承载截止时间，不能填演出开始时刻，也不能
每次刷新都重新延长。演奏者的客户端解析路径会减去 5 秒；要得到同一逻辑
截止时刻，演奏者接收值需加 5 秒补偿，观众接收值无需该补偿。

规定时长由服务器调度策略决定；不得从单份抓包的 `stage=48`、
`stageStart=90` 推导出固定演出时长。

## 演出内容与跨房分发

```text
当前演奏者 → SocialBroadcast（上行 GameMsg 子类型 20）
  → 服务器校验当前演出归属、关卡和会话有效期
  → 向同关卡所有接收房分发
  → SocialBroadcastDown(24)
       sessionId = hash(当前演奏者)
       time / counter / JSON 内容按上传语义保留
```

接收客户端需要先具备演奏者档案和身份绑定，才能把内容挂到正确实体。
跨房转发必须覆盖源房与其他接收房，避免源房提前直发再收到跨房副本造成
重复。管理协议可以独立使用事件 ID、序号和会话轮次保证幂等；不能未经
客户端证据就把 wire counter 当作严格递增序号。

## 结束与下一场

```text
当前演出结束（主动停止 / 规定时长到期 / 演奏者资格失效）
  → 所有接收房获知结束状态
  → 清理旧演奏者状态
  → Reliable AudienceChat(26)：entityId=10145，payload=""
  → 客户端清理活动广播对象
  → 有等待者：提升下一名有效申请者，发布新演出
```

10145 是可靠聊天条目的控制 entityId，不是应用层 packetId；即使控制
字符串为空，也必须编码合法字符串长度。结束状态不能仅依赖座位变化，
即使观众与座位没有变动，也必须传播。

## 不变量

- entityId = bench×8 + seat + 1；普通实体索引上限 10143，10144/10145 为聊天控制 ID。
- 档案 f0 = 身份哈希，是房间成员与演出实体绑定的基础。
- 需要档案的剪影记录应先完成 ReliableData(23) 建立，再发送 Audience(18)。
- 跨房样本必须处于接收端可查询的时钟域，避免未来时间戳让剪影不可见。
- 同一演出使用固定逻辑截止时间；客户端每次重连、快照刷新不能重开计时。
- 新演出开始后不得应用上一场迟到的内容或结束消息。

字段证据与状态机详见 [AudienceStreamInfo](../audience/AudienceStreamInfo.md)、
[SpotlightResponse](../audience/AudienceSpotlightResponse.md)、
[SocialBroadcastDown](../audience/SocialBroadcastDown.md) 和
[AudienceChat](../audience/AudienceChat.md)。
