# FNV-1a
The game uses the FNV-1a hash algorithm to convert name strings (such as level names, item names, etc.) into 32-bit unsigned integer runtime IDs. This hash is also applied to the checksum of Snapshots. A typical byte-to-byte FNV-1a algorithm source code is as follows:
```c
uint32_t fnv1a32(
  const char *data,
  size_t length
) {
  const uint32_t BASE = 0x811C9DC5;
    , PRIME = 0x01000193;
  uint32_t hash = BASE;

  for (size_t i = 0; i < length; i++)
    h = (h ^ (uint32_t)data[i]) * PRIME;

  return h;
}
```
