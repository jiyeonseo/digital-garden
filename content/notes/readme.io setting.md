---
title: "readme.io 세팅"
tags:
- readmeio
---

## API References Try it Authentication  추가하기 
![](https://user-images.githubusercontent.com/2231510/205079888-b4b68d17-aefc-4043-be51-6a94e89e64cd.png)
- `openapi.json` 으로 싱크하는 경우, open api spec 중 `SecurityScheme` 을 맞춰주어야 한다. 
- [Security Scheme Object](https://docs.readme.com/main/docs/openapi-compatibility-chart#security-scheme-object)

### FastAPI에서 추가하기 
```python
from fastapi import FastAPI, Security
from fastapi.security import APIKeyHeader

x_api_key = APIKeyHeader(name="x-api-key", auto_error=False)

async def auth(x_api_key: str = Security(x_api_key)):
  if x_api_key == get_config("API_KEY"):
    return x_api_key
  else:
    raise UnauthorizedException()

router = APIRouter(dependencies= [Depends(auth)])

app = FastAPI()
app.include_router(router)

```

swagger에서도 Authorize가 생긴 것을 볼 수 있다. 
![](https://user-images.githubusercontent.com/2231510/205088702-dc58cc59-75d9-47d1-8215-2329a673400d.png)
`openapi.json` 확인하면 `components.securitySchemes` 가 추가 되어있다.
![](https://user-images.githubusercontent.com/2231510/205089535-3e12ea04-6082-44ff-93f3-ac7ca1e3a491.png)