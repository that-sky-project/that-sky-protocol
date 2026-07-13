# GameMsgHeader

- gameMsgType
  - [GameMsgType](../enums/GameMsgType.md)
- levelSeq
  - u08
- sourcePlayer
  - u08

|field name|note|
|-|-|
|levelSeq|Level change sequence, updated by [LevelUpdatePacket](../LevelUpdatePacket.md). GameMsgPacket with mismatch sequence number will be ignored by the client.|
|sourcePlayer|Player id of the packet original sender. Only valid in GameMsgType_NetRpc.|
