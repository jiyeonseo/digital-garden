---
title: "Redis OM"
tags:
- redis
---
- Object Mapper for Redis
- 인메모리 DB Redis에서도 기존 RDB 라이브러리(SQLAlchemy, Peewee, Django ORM 등)에서와 같이 **declarative model**로 데이터를 다룰 수 있게 해준다. 
- **Pydantic** 사용하여 데이터 유효성 검사 기능 역시 편리하게 사용할 수 있다. 
- Query문과 Secondary indexes 역시 제공. 
- 동기, 비동기(asyncio) 모두 제공.

## base models
- `HashModel` : Hash 형식으로 redis에 값을 저장.
- `JsonModel` : Json object 형식으로 redis에 값을 저장.

```py
import datetime

from typing import Optional

from pydantic import EmailStr

from redis_om import HashModel

class Customer(HashModel):
	first_name: str
	last_name: str
	email: EmailStr
	join_date: datetime.date
	age: int
	bio: Optional[str]

# andrew라는 Customer 객체 생성

andrew = Customer(
	first_name="Andrew",
	last_name="Brookins",
	email="andrew.brookins@example.com",
	join_date=datetime.date.today(),
	age=38,
	bio="Python developer, works at Redis, Inc."
)

# 모델은 unique PK를 자동으로 생성한다.
print(andrew.pk) # (Redis 통신 따로 필요 없음.)
# > '01FJM6PH661HCNNRC884H6K30C'

# `save()` 호출로 Redis에 모델 저장 
andrew.save()

# pk로 Customer 에서 객체를 찾을 수도 있다.
assert Customer.get(andrew.pk) == andrew
```
- 자동으로 생성되는  PK는 unique 하며 정렬 가능하다. (sortable)

### Globally unique primary keys
- Redis OM은 자동으로 유니크 키값을 생성하며 이 값으로 객체를 저장하거나 검색시 사용할 수 있다. 
- 객체 생성시 바로 반환되며 Redis와의 통신은 따로 필요하지 않다. 
- 이렇게 할 수 있는 것은 [[notes/Universally Unique Lexicographically Sortable Identifiers]] 스펙을 따르고 있기 때문이다. 

### Data validation with Pydantic
- 기존 레디스에서는 데이터 스키마를 강제하지 않고 있지만 Redis OM의 Pydantic model을 이용하면 다른 RDB에서와 같이 validation을 체크 할 수 있다. 

```py
try:
	Customer(
		first_name="Andrew",
		last_name="Brookins",
		email="Not an email address!",
		join_date=datetime.date.today(),
		age=38,
		bio="Python developer, works at Redis, Inc."
	)
except ValidationError as e:
	print(e)
	"""
	pydantic.error_wrappers.ValidationError: 1 validation error for Customer
	email
	value is not a valid email address (type=value_error.email)
	"""
```

## References
- [Introducing Redis OM for python](https://redis.com/blog/introducing-redis-om-for-python/)