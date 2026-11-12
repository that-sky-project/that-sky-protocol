# DisconnectPacket (1) ◆

| 字段 | 类型 | 说明 |
|-|-|-|-|
| reason | u8 | |

双端均可发起。ENet 层还有原生 disconnect 事件（带 reason data），应用层
收到此包通常伴随 ENet 断开。不消耗 C→S 序号（见 headers）。
