---
title: "ULID"
tags:
- UUID
- identifier
---
- 128 bits 이며 UUID와 호환 가능
- 매 밀리세컨드마다 1.21e+24 유니크값 생성 가능
- 시간을 나타내는 앞 48bits와 임의값 80bits로 구성된 값. 
```
 01AN4Z07BY      79KA1307SR9X4MV3

|----------|    |----------------|
 Timestamp          Randomness
   48bits             80bits
```
- Timestamp
	- timestamp에서 밀리세컨드 단위로 기록하여 생성 순서대로 정렬할 수 있다.
	- 10889년까지 만들수 있음. 
- 대소문자 구별하지 않음 
- Crockford's Base32 인코딩 (I, L, O, U 제외)
- 
## UUID의 한계점
- [Universally Unique Identifier(UUID)](https://datatracker.ietf.org/doc/html/rfc4122.html)는 중앙 관리식이 아니면서도 유일성을 보장해주는 방식
- 32개의 16진수로 표시되며 총 5개의 version이 있다. 
	- v1 : uniqueness
## References
- [ulid spec](https://github.com/ulid/spec)
- [UUID vs ULID](https://velog.io/@injoon2019/UUID-vs-ULID)
- [How probable are collisions with ULID’s monotonic option?](https://zendesk.engineering/how-probable-are-collisions-with-ulids-monotonic-option-d604d3ed2de)