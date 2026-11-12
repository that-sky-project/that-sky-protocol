# GameMsgType_NetLevelElectionNominee (17) ◆

| 字段 | 类型 | 说明 |
|-|-|-|
| levelId | u32 LE | |
| reason | u16 LE | |

玩家提名自己竞选关卡权威。官方 322 上行 / 74 下行——服务器选择性转发给
同关他人（peers 在 Elect 落地前先知道谁在竞选）。XYSKY UDP 已实现转发。
