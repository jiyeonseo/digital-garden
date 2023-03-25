---
title: "(Build Your Own Redis with C/C++) 3. Implement basic commands"
tags:
- redis
- C
---
> [Build Your Own Redis With C/C++](https://build-your-own.org/redis) 을 보고 정리한 글입니다. 더 자세한 나용은 원문을 참고하실 수 있습니다. 직접 실습한 코드는 [jiyeonseo/build-your-own-redis-with-c-cpp](https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp) 에서 찾아볼 수 있습니다. [[notes/build your own redis with c cpp 2]] 이어지는 노트입니다. 

기본 command 스키마는 아래와 같이 한다. 
```
+------+-----+------+-----+------+-----+-----+------+
| nstr | len | str1 | len | str2 | ... | len | strn |
+------+-----+------+-----+------+-----+-----+------+
```
- `nstr` : number of string. 문자열 갯수. (32 bits)
- `len` : length of the following string. 문자열 길이 (32 bits)

응답은 아래와 같이 32 bits 상태 코드와 응답 메세지로 구성된다. 
```
+-----+---------+
| res | data... |
+-----+---------+
```


## `get`, `set`, `del`
