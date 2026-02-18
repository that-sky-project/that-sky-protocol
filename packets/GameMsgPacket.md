# GameMsgPacket

- gameMsgHeader
  - [GameMsgHeader](./types/GameMsgHeader.md)
- data
  - dependency on `gameMsgHeader.gameMsgType`:
    - if GameMsgType_NetLevelDataHeartbeat
      - [NetLevelDataHeartbeat](./types/NetLevelDataHeartbeat.md)
