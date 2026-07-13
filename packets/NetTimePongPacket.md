# NetTimePongPacket (13)

- requestId
  - u16
- serverRecvTime
  - double
- requestInteval
  - double

|field name|note|
|-|-|
|requestId|The request id of the corresponding NetTimePingPacket.|
|serverRecvTime|The timestamp in millisecond when the server received the NetTimePingPacket.|
|requestInteval|The request inteval of the last NetTimePingPacket from the same client, in millisecond.|
