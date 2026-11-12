# GameMsgType_NetRpc (2) ◆

客户端 MetaClass RPC 通道。信封后即 rpcData——sourcePlayer 在信封里，
rpcData 不含发送者字段。

## 记录链

rpcData 是**多条记录串联**，每条记录头：

| 字段 | 类型 | 说明 |
|-|-|-|
| classHash | u32 LE | MetaClass 哈希（客户端 sub_1A692A8：u32+u8+bounded u16） |
| flags | u8 | 逐类语义（gift 抓包恒 0x00） |
| bodyLen | u16 LE | ≤ 0x1000 |
| body | u8[bodyLen] | 类自定义格式 |

记录循环直到**整段精确耗尽**（exact-consume）。**未识别的记录逐字节
透传**——客户端自己按哈希路由。

## 两种线格式（自同步解析）

部分记录在 classHash 前还带 **1 字节 slot**（关联玩家 id），由此产生两种
线格式：

| 线格式 | 单条记录 | 证据 |
|-|-|-|
| **no-slot** | `[classHash u32][flags u8][bodyLen u16][body]`（7B 头） | 实测客户端直发流量：12B 单记录（hash 0xd71867be, bodyLen 5）精确耗尽；880B = 110 条聊天记录（hash 0x5428fb22）链式精确耗尽 |
| **slot** | `[slot u8][classHash u32][flags u8][bodyLen u16][body]`（8B 头） | 官方抓包 gift/chat 记录（hash 前多 1 字节玩家 id：0x1d/0x1f/0x83，中继记录） |

解析规则：先按 no-slot 链解析、**精确耗尽即接受**；否则按 slot 链重试；
两条链都不耗尽则整段原样透传。classHash 本身足以路由记录类别，与是否
带 slot 无关——下游一律按 **body 相对偏移**处理，即对两种框架一致。

## 礼物 RPC（classHash 0x20d9b9b3 = 字节 b3 b9 d9 20）

body 布局（DOWN 83B 官方样本，slot 框架）：

| body 偏移 | 字段 |
|-|-|
| 0 / 1 | 发送者 / 接收者槽位 |
| 2 | type：0=offer 1=**accept** 2=complete 3=cancel |
| 4..11 | 能力 id、数量等 |
| **12..27** | receiver uuid u8[16]（绝对偏移随框架：slot 头 +8 / no-slot 头 +7） |

accept（body[2]=1）时服务器把**接受者本人 uuid** 写入 12..27（防伪造）；
offer / complete / cancel 原样透传。

验证：**[capture]** UP 51B / DOWN 83B / 双记录 102B 逐字段 +
**[impl]**（实机日志 no-slot 链精确耗尽）+ **[asm]** sub_1A94BBC（gift）/
sub_1A692A8（记录头三元组）。
