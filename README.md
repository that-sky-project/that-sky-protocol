# That Sky Protocol

Sky: Children of the Light 协议文档总集。分两个通道：

| 通道 | 内容 | 文档 |
|-|-|-|
| **HTTP(S)** | 账号/货币/拾取等 REST 请求 | [requests/](requests/) |
| **UDP（ENet 可靠）** | 实时游戏会话：入房、状态同步、关卡权威、跨房迁移、千人房观众 | [packets/](packets/README.md) |

**入口**：UDP 协议的架构、会话流程、包索引见 [packets/README.md](packets/README.md)。
