# MoveResultPacket (10) ▲

客户端对迁移指示的回执。

| 字段 | 类型 | 说明 |
|-|-|-|-|
| result | u8 | **0 = 接受**（将断开重连）；非 0 = 拒绝/失败原因 |
| trailing | 原样 | 服务器解出但不消费 |

服务器处理：`result==0` → `MOVING→(重连)`；非 0 → `MOVING→ACTIVE` +
权威 rebalance。超时 20s（`move_timeout_ms`）强制恢复。

验证：**[capture]** + **[impl]**。
