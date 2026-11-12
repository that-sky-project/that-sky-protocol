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

payload 逐字节扫描：非 0 字节原样；**0x00 后跟计数 N 表示 N 个零**。
例如 `00 01` 展开为一个零，`00 03` 展开为三个零。历史文档的 N−1
说法误读了原生解码器向量循环的边界；标量循环 N=1 时仍写入一个零。

checksum 计算范围是线上帧头后三字节以外的 payload：差分帧校验压缩后的
payload，不校验解压或 XOR 后的状态。XOR 差分压缩后反而变大时使用 raw。

0.35.1 ：发送 `sub_2A1ACFC` 的 `0x2A1AF6C/70` 对每个输入零
递增 run，`0x2A1AF08/10` 输出原 run；接收 `sub_2A1B11C` 的
`0x2A1B370..380` 执行 N 次写零。校验路径为 `0x2A1AFBC..AFE4` 与
`0x2A1B3EC..B414`。

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
