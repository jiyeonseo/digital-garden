---
title: "FastAPI로 reverse proxy server 만들기"
tags:
- python
- fastapi
---
[[notes/Rate Limiter]] 와 같이 중간에 Auth, Rate Limit, IP Ban 등 기능을 하는 중간 Gateway 서버로 사용시 아래와 같이 만들 수 있다.

```python
import httpx
from fastapi import FastAPI, Request

from starlette.requests import Request
from starlette.responses import StreamingResponse
from starlette.background import BackgroundTask

app = FastAPI(docs_url=None, redoc_url=None, openapi_url=None)

proxy_base_host = "localhost"
proxy_base_port = 8000
client = httpx.AsyncClient(base_url=f"http://{proxy_base_host}:{proxy_base_port}")

async def _reverse_proxy(request: Request):
    url = httpx.URL(path=request.url.path,
                    query=request.url.query.encode("utf-8"))
	_headers = {
		**request.headers,
		"host" : proxy_base_host
	}
    rp_req = client.build_request(request.method, url,
                                  headers=_headers,
                                  content=await request.body())  
    rp_resp = await client.send(rp_req, stream=True) 
    return StreamingResponse(
        rp_resp.aiter_raw(),
        status_code=rp_resp.status_code,
        headers=rp_resp.headers,
        background=BackgroundTask(rp_resp.aclose),
    )

app.add_route("/{path:path}",
              _reverse_proxy, ["GET", "POST"])
```

## References
- [Can fastapi proxy another site as a response to the request?](https://github.com/tiangolo/fastapi/issues/1788)