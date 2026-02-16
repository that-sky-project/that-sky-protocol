# RequestHeader
Merged with the request.

|key|type|note|
|-|-|-|
|user|string|Player uuid|
|user_id|string|Usually the same as `user`|
|session|string|Session id, 16 byte hex.|

# ResponseHeader
Merged with the response.

|key|type|note|
|-|-|-|
|session|string|Session id, 16 byte hex. The same as the one of the request.|

