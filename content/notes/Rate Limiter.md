---
title: "처리율 제한 장치 Rate Limitter"
---

## 필요한 이유
- 일정 시간 당 요청수를 제한함.
	- e.g. RPS (Request Per Second)
- 서비스에 한 번에 엄청난 수의 요청을 받게 된 경우 서버를 안정적으로 운영하기 위해 특정 요청에 대해서만 허용하는 방법. 요청의 임계치를 정하여 그 값을 초과하게 되면 처리하지 않는다. 
- 특히, DDOS와 같이 불필요한 요청을 하여 시스템을 공격을 방어하기 위해 사용.
- 수용 가능한 요청에 대해 처리하며, 이로 인해 서버 리소스 비용을 조절할 수 있게 됨으로써 비용 절감의 효과도 얻을 수 있다.

아래와 같이 실제 서비스에 닿기 전 유효한 요청인지 받을 수 있을지에 대해 Rate Limiter가 판단 후  실제 서비스로 요청이 갈 수 있도록 한다. 

![](https://miro.medium.com/max/720/1*maqXnVMCWj_Z28qfmB_ZgQ.webp)
( ref : https://towardsdatascience.com/designing-a-rate-limiter-6351bd8762c6 )

너무 많은 요청이 와서 이를 막을 경우 [HTTP status 429 Too Many Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)를 반환한다. 이러한 중간 역할을 하는 장치를 주로 API Gateway라고 부르는데, 처리율 제한 뿐만 아니라 SSL 종단(termination), 인증(Autherntication), IP 차단/허용(deny/allow list) 등을 지원하는데, 클라우드 프로바이더들이 제공하는 managed를 사용할수도 있고, 직접 만들어 사용할 수도 있다. 

## 처리율 제한 알고리즘
### 토큰 버킷 알고리즘 

### 누출 버킷 

### 고정 윈도 카운터

### 이동 윈도 로그

### 이동 윈도 카운터

## 처리율 제한 규칙

### 1) 시간당 처리율 
- RPS, PRM 등 시간 내 요청 수 제한


### 2) Compute Unit 
- API endpoint 마다 정보를 쿼리하는데 필요한 계산에 따라, 각 API 요청에 대한 가중치를 부여하는 방법. 이 단위를 Compute Unit 줄여 CU라고 부른다. 
- CUPS (Compute Unit Per Second) 

## Response
- API 요청 입장에서도 얼마나 나의 limit이 남았는지, 그리고 초과되었다면 언제 부터 다시 요청이 가능한지 알수 있으면 더 효율적으로 API 요청을 할 수 있다. 
- `x-ratelimit-limit` : 윈도우 마다의 제한 수 
- `x-ratelimit-remaining` : 현재 윈도우 내 남은 요청 제한 수 
- `x-ratelimit-retry-after` : 요청 제한이 넘어간 경우, 얼마 뒤(대부분 "초") 다시 요청을 보내야하는 지

## References
- [Designing a rate limiter](https://towardsdatascience.com/designing-a-rate-limiter-6351bd8762c6)
- [가상 면접 사례로 배우는 대규모 시스템 설계 기초](http://www.yes24.com/Product/Goods/102819435)
- [Examples of HTTP Rate Limiting Response headers](https://stackoverflow.com/questions/16022624/examples-of-http-api-rate-limiting-http-response-headers)
- [What is a compute unit(CU)?](https://moralis.io/faq/what-is-a-compute-unit-cu/)