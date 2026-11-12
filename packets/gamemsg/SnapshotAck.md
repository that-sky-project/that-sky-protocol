# GameMsgType_SnapshotAck (14) ◆

| 字段 | 类型 | 说明 |
|-|-|-|
| playerAckSeq | u8 | 已确认的 PlayerState 快照序号 |
| levelAckSeq | u8 | 已确认的 NetLevelData 快照序号 |

服务器用它裁剪出站快照窗口（保留 `(seq−ack)&0xFF <= 127`）。缺省 1。

验证：**[capture]** 官方样本 `0e01000101`（双向）。
