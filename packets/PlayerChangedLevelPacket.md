# PlayerChangedLevelPacket (8) ▼

| 字段 | 类型 | 说明 |
|-|-|-|-|
| playerId | u8 | 换图玩家的 netId |
| levelId | u32 LE | 新关卡 id |

服务器在某玩家 LevelUpdate 成功后广播全房。

验证：**[capture]** 官方样本 `5a 09d2f55c`（连续 4 个，levelId 递增帧位）。
