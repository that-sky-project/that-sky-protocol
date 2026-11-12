# 位流原语

Sky 的序列化是**位级**的：compressed/bool/量化浮点按位读写，整数/浮点
按字节读写且**自动对齐**。两种模式的关键规则：

## 位原语（不对齐）

| 原语 | 位宽 | 说明 |
|-|-|-|
| bool | 1 | |
| compressed(max) | ceil(log2(max+1)) | 无符号，值域 [0,max]。`compressed(10143)`=14bit，`compressed(15)`=4bit，`compressed(7)`=3bit |
| bounded(min,max) | ceil(log2(max−min+1)) | 有符号偏移：wire = value − min。**signed bounded(-1,10203)** 的 wire = stored+1（StreamInfo 索引字段易错点） |
| 量化浮点 | n bit | 线性映射 [min,max]；pivot 双段非线性（旋转 ±π/6bit、速度 ±150/8bit） |

位宽公式（客户端 sub_2955944）：`bits = ceil(log2(max−min+1))`，
span=7→3bit、span=15→4bit、span=10143→14bit、span=10204→14bit。

## 字节原语（自动对齐）

| 原语 | 字节 | 说明 |
|-|-|-|
| u8 | 1 | |
| u16 / u32 / u64 | 2/4/8 | LE；**读取前若位未闭合，先跳到下一字节**（sub_2954130 的 +32 标志逻辑），与节点 BinaryReader.alignToByte 一致 |
| f32 / f64 | 4/8 | LE，同上自动对齐 |
| memcpy n | n | 指针直读（如位置块 6 字节、PlayerState 条目） |

**这是 id 18 帧头"192 bit"的由来**：idxStart(14)+idxEnd(14)=28 bit，
f64 对齐到 32，+64 = 96；origin 3×f32 = 96 → 头部共 192 bit = 24 字节。
