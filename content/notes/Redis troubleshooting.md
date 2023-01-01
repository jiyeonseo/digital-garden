---
title: "Redis 트러블슈팅"
tags:
- redis
---

## "WRONGTYPE Operation against a key holding the wrong kind of value"
- Redis는 [여러 데이터 타입](https://redis.io/docs/data-types/)을 제공한다. 각각에 맞는 명령어를 써 주어야 하는데 실제 키 값의 타입과 명령어가 맞지 않으면 이와 같은 에러가 난다. 
- 실제 value의 타입과 명령어가 일치하는지 확인 필요
	- value 타입 확인 명령어 : `type <key>`

### 타입별 값 가져오는 명령어 
- type `string` -> `GET <key>`, `MGET <key>`
	- [INCR](https://redis.io/commands/incr/) 은 string 값도 integer로 파싱하여 +1 해준다. 
- type `hash` -> `HGET <key>` , `HMGET <key>`
- type `list` -> `Irange <key> <start> <end>`
- type `sets` -> `smembers <key>`
- type `sorted sets` -> `ZRANGEBYSCORE <key> <min> <max`

### References
- [WRONGTYPE Operation against a key holding the wrong kind of value php](https://stackoverflow.com/questions/37953019/wrongtype-operation-against-a-key-holding-the-wrong-kind-of-value-php)