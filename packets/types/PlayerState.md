# PlayerState（0.35.1）

玩家完整状态序列化。布局随客户端构建变化——**同一 0.35.1 存在两种已知
变体**：state+0x52B8 处 8 字节字段的有无，导致其后所有字段移位 64 bit。
客户端运行时探测布局：逐候选"试吃整条记录"，恰好消费完（剩余 ≤7 bit）
者胜；成功布局会被记忆，后续帧优先尝试。

## 字段总表（按线序）

| 字段 | 类型 | 客户端引用 |
|-|-|-|
| stateKind | bounded(0,4) | state+0xb0 |
| position | f32 ×3 | state+0x5990 |
| net | struct | sub_140E7F910 |
| moveHeader | struct | sub_140E803C0 |
| offset_52cb / 52b0 / 519c | u8 / u16 / u32 | 布局分界 |
| **offset_52b8** | u8（变体 A）/ 无（变体 B） | state+0x52b8——两种构建的分水岭 |
| move | struct（速度块） | sub_140C9E230：rotation ±π 6bit、velocity ±150 8bit、direction、scalar ±20 5bit、flag |
| body | BodyKeyFrame[0..4]，stride 0xB0 | 斗篷/道具布料链 |
| queuedState | struct | state+0x55a0 |
| poseMode | bounded(0,3) | state+0x5b71 |
| levelMeshId | u32 | state+0x5b48 |
| collideOffset | f32 ×3 | state+0x5a10 |
| action / energy | struct | 表情动作 / 斗篷能量与飞行 |
| offset_5a20 | f32 ×3 | |
| collideDirection | 量化 vec3（−1..1, pivot 0, 5bit） | |
| anim | struct | |
| candle | struct（4 bool + PlayerId + qf） | 烛光 |
| follower | struct（f32 + PlayerId + bool + 2 bounded） | 跟随 |
| lookTarget | struct（gated bool + PlayerId + vec3） | |
| offset_5198 | 量化 f32(0..50, 8bit) | |
| offset_52c0 | sbounded(−1..32) | |
| offset_52c4..52d0 | mixed（u16/u8/bounded/bools） | |
| offset_5b30 | PlayerId | |
| pose block | u8 / bools / 6×vec3 | |
| outfit | OutfitItem[10] | hair/hat/horn/mask/face/neck/body/feet/wing/prop，含 tintA/B/C 与名称 |
| offset_5b58..5b44 | mixed（u32/u8/bounded(0,255)/PlayerId） | |
| buffs | [0..5] × (u8 id + u8 value) | |
| offset_5b64..5ac0 | mixed（u32/u16/PlayerId/bool/qf） | |
| offset_5b7d | PlayerId[0..7]（bounded count + ids） | |
| social | struct | 牵手/拥抱/肩背 |
| worldQuests | 固定 0x100 bool | |
| tail | struct（slotTable 四趟交错遍历） | sub_140C9ED50 |
| trailer | mixed（u32 / 16B blobs / flags / vec3 / 32B / 10×u8） | |

## 前缀稳定性（消费兼容性）

前 **13 字节**在所有已知版本中稳定：`stateKind = b[0] & 7`、
`position = 3×f32 @1,5,9`。任何解析失败都以它降级，位置数据不丢。

## 关键怪癖

- **位流中混有必须字节对齐的定长字段**——把整条记录当纯位流处理会导致
  漂移、不可解码。
- PlayerId::Serialize 三模式：裸 u8 / u8 net-id 槽（0.35.1 上传路径固定
  mode 2）/ 裸 u32。
- 记录上限 0x4B5 字节。

验证：**[asm]** sub_140C9F280 全字段 + **[capture]** 双布局实跑 +
**[impl]**。
