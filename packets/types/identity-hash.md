# 身份哈希（FNV-1a-64 of UUID 原始字节）

观众系统贯穿使用的玩家指纹：

```
hash = 0xCBF29CE484222325
for each byte b of (uuid 的 16 个 wire 字节):
    hash = (hash ^ b) * 0x100000001B3  (mod 2^64)
```

**是对 16 字节二进制做哈希，不是对 uuid 文本**。黄金向量（`foobar` →
0x85944171f73967e8）验证实现与客户端 sub_1157808/sub_10FE178/sub_115BA7C
完全一致。

| 用途 | 匹配对象 |
|-|-|
| RoomIds (22) 成员绑定 | 档案 f0 == hash(本房玩家) |
| StreamInfo (28) 演奏者判定 | sessionId == hash(自己) → 状态 3 |
| SocialBroadcastDown (24) 动作绑定 | sessionId == hash(演奏者)；== hash(自己) 则丢弃 |
| 档案识别 (23) | f0==0 → 客户端 LCG(entityIdx) 随机外观 |

标准测试向量：`foobar` → `0x85944171f73967e8`。
