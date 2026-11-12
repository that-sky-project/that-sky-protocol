# GameMsgHeader

| 字段 | 类型 | 说明 |
|-|-|-|
| gameMsgType | u8 | [GameMsgType](../enums/GameMsgType.md) |
| levelSeq | u8 | levelChangeCount；不匹配接收方本地值 → 整包丢弃（观众族请求例外） |
| sourcePlayer | u8 | 原始发送者 netId；C→S 由服务器改写为转发者 |
