# SnapshotResyncPacket (5) ▼

**零载荷**命令。客户端 id-5 分支（sub_1A99E98）无条件：

1. 重置 PlayerState 快照 Writer（窗口重置，容量参数 10）
2. 重置 PlayerState 快照 Reader
3. 重置 NetLevelData 快照 Writer
4. 重置 NetLevelData 快照 Reader
5. 写 `u16 0x0101` 到 `state+0x2E3C0`（两通道"需要 keyframe"标志）
6. 通知关卡层（sub_17A2164）

服务器触发时机：某会话的出站快照窗口 `resyncfailed`（16 槽窗口两次对半
收缩仍无 ack）——此后任何增量都无法恢复，必须让客户端整体重置并索要
raw keyframe。XYSKY UDP 在 每帧检查会话快照窗口状态并发送。

旧协议表把它标成 `EnterGamePacket_DEPRECATED`——0.35.1 中该槽位已
复活为此命令。

验证：**[asm]**（逐指令还原）+ **[impl]**（触发链路单测）。
