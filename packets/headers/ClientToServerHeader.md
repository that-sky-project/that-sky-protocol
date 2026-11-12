# ClientToServerHeader

每个 C→S ENet payload 以 6 字节定长头开始。

| 偏移 | 字段 | 类型 | 说明 |
|-|-|-|-|
| 0 | token | u32 LE | **connMagic**：客户端生成的会话凭据。JoinGame 时首次上报，服务器绑定到会话；此后每包必须一致，否则按坏包丢弃（`magic_mismatch`）。 |
| 4 | packetId | u8 | [PacketIds](../enums/PacketIds.md) |
| 5 | sequence | u8 | 包序号。每发一个包 +1，u8 环绕；**id 1 (Disconnect) 与 id 2 (Kick) 不消耗序号**（客户端发送原语 sub_1A72F24 的 `(a2 & 0xFD) != 1` 分支）。 |

验证：**[asm]** sub_1A72F24（发送组帧）+ **[impl]**（节点 connMagic 校验）。
