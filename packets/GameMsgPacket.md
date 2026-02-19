# GameMsgPacket

- gameMsgHeader
  - [GameMsgHeader](./types/GameMsgHeader.md)
- data
  - dependency on `gameMsgHeader.gameMsgType`:
    - if GameMsgType_NetLevelDataHeartbeat
      - [NetLevelDataHeartbeat](./types/NetLevelDataHeartbeat.md)
    - if GameMsgType_PlayerState
      - snapshotHeader
        - [SnapshotHeader](./types/SnapShotHeader.md)
      - playerStates
        - **example element**
          - size
            - u32
          - playerId
            - [PlayerId](./types/PlayerId.md)
          - state
            - [PlayerState](./types/PlayerState.md)
