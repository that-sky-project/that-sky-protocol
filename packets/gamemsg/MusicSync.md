# GameMsgType_MusicSync (15) ◆

音乐同步，**不透明转发**。观测结构（不参与转发逻辑）：
`[u8 flags][u32 len][轨道名\0][f64 进度]…`，轨道名如 `AP23QuestDrama02`、
`DoMischief25Music`。旧观测工具按"32B 定长 C 字符串"解析会错位——只影响
调试输出，转发路径不受影响。官方 66058 下 / 6594 上。
