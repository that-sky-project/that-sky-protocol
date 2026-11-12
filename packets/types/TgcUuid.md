# TgcUuid

| 字段 | 类型 |
|-|-|
| bytes | u8[16] |

文本形式 `8-4-4-4-12` hex。**观众的 identityHash 就是对这 16 个 wire
字节做 FNV-1a-64**（见 [identity-hash.md](identity-hash.md)）。
