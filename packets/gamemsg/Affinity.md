# GameMsgType_Affinity (7) ▲

玩家间好感度上报（观众系统的 affinity 遥测源）。

| 字段 | 类型 |
|-|-|
| count | compressed(0,8) |
| entries ×count | netPlayerId u8、value u8（value/255 归一化） |

服务器存 `player.lastAffinity`，进入 QWD 遥测，供迁移规划器的 social
信号使用。验证：**[capture]** 空样本 `07 01 00 | 00`。
