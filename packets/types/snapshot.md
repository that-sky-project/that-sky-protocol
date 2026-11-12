# 快照差分（SnapshotHeader + 窗口）

PlayerState（GameMsg 3）与 NetLevelData（GameMsg 8/11）共用一套差分机制。

## 帧头（3 字节）

| 字段 | 类型 | 说明 |
|-|-|-|
| save_seq | u8 | 本帧序号（0 = delta 帧，不入窗） |
| base_seq | u8 | 引用基帧序号（0 = raw 全量帧） |
| checksum | u8 | **fnv1a8**：`hash=0xC5; hash=(0x93*(hash^b))&0xFF` 逐字节 |

帧型：
- **raw**：save≠0, base=0 — 全量原文，入窗
- **keyframe**：save≠0, base≠0 — XOR 差分 + RLE，入窗
- **delta**：save=0, base≠0 — XOR 差分 + RLE，不入窗

## RLE

payload 逐字节扫描：非 0 字节原样；**0x00 后跟计数 N 表示 N−1 个零**。
XOR 差分后反而变大则放弃差分直接 raw。

## 窗口（16 槽）

- 初始种子：sequence=1 的空快照，nextSaveSeq=2（对齐原生 Initialize）
- 每 keyframeInterval（默认 10）帧发一个 save 帧；resync 期间减半
- ack(seq)：保留 `(seq−ack)&0xFF <= 127` 的条目（u8 环绕安全）
- 窗口满（16）→ resync：清窗，下一帧 raw，间隔减半；**已处于 resync
  再满 → `resyncfailed`**（窗口砍半）→ 服务器发 id 5 强制重置

## 客户端解码状态机

`hasherror`（校验不符）/ `resyncing`（等 keyframe 期间收 delta 忽略）/
`resyncstart`（base 缺失清窗）/ `resyncsuccess`——四个状态与节点
SnapshotReader 一一对应。

验证：**[capture]**（35453 下行帧 + 1829B 双向样本逐字节）+ **[asm]**
（校验和常数 0xC5/×147、RLE 语义、16 槽 0x2108B 窗口全部对上）。
