# EnterGamePacket (17)

Updates the player(s) in the room. Sent by the server when new player connects.

- playerCount
  - compressed
- playerList
  - **example element**
    - playerNetId
      - u08
    - playerUuid
      - [TgcUuid](./types/TgcUuid.md)
    - \<unknown\>
      - u08\[16\]
    - levelId
      - i32

|field name|note|
|-|-|
|levelId|The FNV-1a hash of level name.|
