---
title: "Redis OM"
tags:
- redis
---
- Object Mapper for Redis
- 인메모리 DB Redis에서도 기존 RDB 라이브러리(SQLAlchemy, Peewee, Django ORM 등)에서와 같이 **declarative model**로 데이터를 다룰 수 있게 해준다. 
- **Pydantic** 사용하여 데이터 유효성 검사 기능 역시 편리하게 사용할 수 있다. 
- Query문과 Secondary indexes 역시 제공한다. 
- 
## References
- [Introducing Redis OM for python](https://redis.com/blog/introducing-redis-om-for-python/)