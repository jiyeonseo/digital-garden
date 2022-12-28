---
title: "Redis OM"
tags:
- redis
---
- [Object Mapper for Redis and Python](https://github.com/redis/redis-om-python)
- high-level 추상화로 데이터 모델과 쿼리를 코드로 편하게 할수 있도록 도와주는 라이브러리
- 인메모리 DB Redis에서도 기존 RDB 라이브러리(SQLAlchemy, Peewee, Django ORM 등)에서와 같이 **declarative model**로 데이터를 다룰 수 있게 해준다. 
- **Pydantic** 사용하여 데이터 유효성 검사 기능 역시 편리하게 사용할 수 있다. 
- Query문과 Secondary indexes  제공. 
- 동기, 비동기(asyncio) 모두 제공.
- Python 뿐만 아니라 [.NET](https://github.com/redis/redis-om-dotnet), [Node.js](https://github.com/redis/redis-om-node), [Spring/Java](https://github.com/redis/redis-om-spring) 등 다양한 라이브러리를 제공하고 있다. 

## 설치하기 
```sh
# With pip
$ pip install redis-om

# Or, using Poetry
$ poetry add redis-om
```

## Redis 연결
```py
# Model마다 redis connection을 연결하는 경우
Customer.Meta.database = get_redis_connection(url="redis://localhost:6379",
                          decode_responses=True))

# 전체 같은 redis connection인 경우 
redis = get_redis_connection()
Migrator(redis).run()

```

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
- async 는 `aredis` module에서 import 받아야 한다. 
```py
# synchronously
from redis_om import HashModel
from redis_om import get_redis_connection

# asynchronously
from aredis_om import HashModel, NotFoundError  
from aredis_om import get_redis_connection
```
- 저장하게 되면 다음과 같이 저장된다. 
	- Key :  `:{package}:{class_name}:{pk}`
	- Value : pk, 정의된 field들
![](https://user-images.githubusercontent.com/2231510/209757427-59aab035-d09c-4332-ae86-32375257108f.png)

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

### Query expressions
- ORM의 또다른 강력한 무기는 API를 이용한 쿼리이다. Redis OM 역시 [RediSearch](https://redis.com/modules/redis-search/) 를 이용하여 Redis에서도 DB처럼 쿼리 및 인덱싱을 할 수 있도록 해준다. 
```py
from redis_om import get_redis_connection

class Customer(HashModel):
	first_name: str
	last_name: str = Field(index=True) # 인덱싱할 키를 정해주고 
	email: EmailStr
	join_date: datetime.date
	age: int = Field(index=True)
	bio: Optional[str]

# last name이 "Brookins" 인 모든 customers
Customer.find(Customer.last_name == "Brookins").all()

```
- async 인 경우는 아래와 같이 `aredis_om` module을 사용하며, 값을 가져올 때 `await`을 사용한다. 
```py
from aredis_om import HashModel

class Customer(HashModel):
	first_name: str
	...

await Customer.all_pks()
```
-


### Embedded model
- `HashModel`에서는 List, Set, Hash와 같은 다른 타입은 사용할 수 없다
- `JsonModel` 에서는 가능하다. 
```py
from redis_om import EmbeddedJsonModel, JsonModel, Field

  
class Address(EmbeddedJsonModel):
	address_line_1: str
	address_line_2: Optional[str]
	city: str = Field(index=True)
	state: str = Field(index=True)
	country: str
	postal_code: str = Field(index=True)

class Customer(JsonModel):
	first_name: str = Field(index=True)
	last_name: str = Field(index=True)
	email: str = Field(index=True)
	join_date: datetime.date
	age: int = Field(index=True)
	bio: Optional[str] = Field(index=True, full_text_search=True, default="")
	address: Address

# "San Antonio, TX"에 사는 모든 customers 구하기 
Customer.find(Customer.address.city == "San Antonio",
				Customer.address.state == "TX")
```

### 다른 Redis 명령어 사용하기 
```py
from redis_om import HashModel

class Demo(HashModel):
    some_field: str

redis_conn = Demo.db()
redis_conn.sadd("myset", "a", "b", "c", "d")

print(redis_conn.sismember("myset", "e")) # False
print(redis_conn.sismember("myset", "b")) # True
```

## References
- [Introducing Redis OM for python](https://redis.com/blog/introducing-redis-om-for-python/)
- [Introducing the Redis OM Client Libraries](https://redis.com/blog/introducing-redis-om-client-libraries/)
- [Redis OM Python](https://redis.io/docs/stack/get-started/tutorials/stack-python/)
- [FastAPI Integration](https://github.com/redis/redis-om-python/blob/main/docs/fastapi_integration.md) - asynchronous example code
- [Flask Integration](https://github.com/redis-developer/redis-om-python-flask-skeleton-app) - synchronous example code