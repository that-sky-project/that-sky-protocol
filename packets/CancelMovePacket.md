# CancelMovePacket (11) ◆

| 字段 | 类型 | 说明 |
|-|-|-|-|
| reason | u8 | 服务器回滚时下发 reason=2 |

服务器侧：组队迁移中途失败（部分成员 Prepare 失败）→ 向已发过 MoveGame
的成员发 CancelMove → 全组 `MOVING→ACTIVE`。客户端上行的 CancelMove 在
0.35.1 服务器未注册 handler（潜在 TODO，见 audit 报告）。

验证：**[capture]**（结构）+ **[impl]**（服务器发送路径）。
