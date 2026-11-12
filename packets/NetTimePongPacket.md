# NetTimePongPacket (13) ▼

| 字段 | 类型 | 说明 |
|-|-|-|-|
| requestId | u16 | 回应的 Ping 序号 |
| serverTime | f64 LE | 服务器 unix 秒（客户端时钟基准） |
| reserved | u32 | 官方恒 0 |
| scalar | f32 | ≈1.30–1.56，逐包变化；官方语义不明，按不透明值回显即可 |

验证：**[capture]** 官方样本 `0000 740e4b0f24b9da41 00000000 78eab33f` →
serverTime=1793364029.17, scalar=1.4056。
