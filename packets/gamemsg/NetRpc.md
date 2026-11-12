# GameMsgType_NetRpc (2) ◆

客户端 MetaClass RPC 通道。信封后即 rpcData——sourcePlayer 在信封里，
rpcData 不含发送者字段。

## 记录格式（0.35.1 抓包铁证）

rpcData 是**多条记录串联**，每条：

| 字段 | 类型 | 说明 |
|-|-|-|
| slot | u8 | 关联玩家槽位（C→S = 发送者） |
| classHash | u32 LE | MetaClass 哈希（客户端 sub_1A692A8：u32+u8+bounded u16） |
| flags | u8 | 逐类语义（gift 抓包恒 0x00） |
| bodyLen | u16 LE | ≤ 0x1000 |
| body | u8[bodyLen] | 类自定义格式 |

记录循环直到缓冲区耗尽。**未识别的记录逐字节透传**——客户端自己按哈希
路由。

## 礼物 RPC（classHash 0x20d9b9b3 = 字节 b3 b9 d9 20）

body 布局（DOWN 83B 官方样本）：

| body 偏移 | 字段 |
|-|-|
| 0 / 1 | 发送者 / 接收者槽位 |
| 2 | type：0=offer 1=**accept** 2=complete 3=cancel |
| 4..11 | 能力 id、数量等 |
| **12..27** | receiver uuid u8[16]（= rpcData 偏移 20） |

验证：**[capture]** UP 51B / DOWN 83B / 双记录 102B 逐字段；**[asm]**
sub_1A94BBC。
