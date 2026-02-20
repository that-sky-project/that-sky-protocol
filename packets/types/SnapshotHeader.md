# SnapshotHeader
A simple compress algoritm.

- sequence
  - u08
- base
  - u08
- checksum
  - u08

|field name|note|
|-|-|
|sequence|Identify the sequence number of the snapshot obtained after decoding the current packet (0 when the current frame is a key frame, and the incremental frame is the sequence number of the new snapshot).|
|base|The incremental frame refers to the sequence number of the reference key frame (0 when the current frame is a key frame).|
|checksum|The per-byte FNV-1a hash of trailing data.|
