---
title: "unleash"
tags:
- deployment
- feature toggle
---
- 피쳐 토글 서비스 (비슷한 서비스로는 [[notes/LaunchDarkly]] 가 있다.)  
- 코드 배포와 기능 배포를 서로 분리시킴으로써 더욱 안전한 배포 유연한 기능 오픈/클로즈를 수행할 수 있다. 
- Client (Server Side) / Frontend (Web+Mobile) 각각의 SDK를 잘 지원하고 있어 여러 방면에서 사용하기 좋다. 
![](https://docs.getunleash.io/assets/images/unleash-architecture-40c7293e777a7bea0df4edeeef8a6bf4.svg)
- Enterprise와 opensource 둘 다 제공하고 있어 입맛에 알맞게 사용할 수 있다. 
	- [Dockerfile](https://docs.getunleash.io/reference/deploy/getting-started)
	- [Helm chart](https://github.com/unleash/helm-charts/)
	
## Feature 토글 타입
- Unleash에서는 5가지 토글 타입을 제공하고 있다. ([doc](https://docs.getunleash.io/reference/feature-toggle-types))
- 토글 타입별로 기능차이는 없으며 표현되는 **아이콘 모양만 다르고** 각각의 **예상 수명시간**이 다를 뿐이다. 
	- 토글 타입을 정했더라도 나중에 변경 가능하다. 
	- [타입별 예상 수명시간](https://www.getunleash.io/blog/feature-toggle-life-time-best-practices) 
- Feature 토글 타입
	- Release : trunk based 환경에서 CD에서 계속해서 배포시 (예상 수명 : 40 days)
	- Experiment : 여러 변수를 사용하는 경우 혹은 A/B 테스팅(예상 수명 : 40 days)
	- Operational :  운영 측면에서 사용하는 경우 (예상 수명 : 7 days)
	- Kill switch : 천천히 기능 삭제해나갈때. (예상 수명 : 영구) - [kill switch best practice](https://www.getunleash.io/blog/kill-switches-best-practice)
	- Permission : 특정 유저들에게만 기능 혹은 프러덕트를 열어주는 경우 (예상 수명 : 영구)
- `stale` 로 mark 해주면 더 이상 사용하지 않는 토글로 표시해둘 수 있다. 
	- 바로 삭제하게 되면 혹시나 예상치 못한 장애로 이어질 수 있지만 이렇게 먼저 표시를 해두게 되면 "health" 탭에서 stale 된 토글이 현재 얼마나 들어오고 있는지 확인할 수 있다. 
	- [Technical Debt](https://docs.getunleash.io/reference/technical-debt) 이 기능이 깨끗한 코드를 유지하는데 큰 도움이 될 것 같다.

![](https://user-images.githubusercontent.com/2231510/208935165-cda5e879-5848-47e7-a15a-357c8e5daae8.png)

## API Token 만들기 
- [How to create API Tokens](https://docs.getunleash.io/how-to/how-to-create-api-tokens)
- 연결 코드에서 사용할 API Token을 먼저 만들어야 한다.
- 서버사이드는 Type `Client`, 모바일이나 웹은 Type `Frontend`, Unleash 자체에 대한 API를 위해서는 Type `Admin` 으로 만들면 된다.
- environment 마다 token을 발행하여 환경별로 기능을 조절할 수 있다. 
![](https://user-images.githubusercontent.com/2231510/208928220-536afe05-ba09-4875-8dc4-6bb8d223f4c7.png)
- 생성하면 SDK에서 연결시 사용해야하는 "API URL"이 나온다. 
	- managed를 사용하게되면 `https://us.app.unleash-hosted.com/**/api/` 

## SDK 연결하기 - Python
- [Python SDK](https://docs.getunleash.io/reference/sdks/python)
```py 
from UnleashClient import UnleashClient  
  
client = UnleashClient(  
url="`https://us.app.unleash-hosted.com/**/api/",  # 위에서 받은 API URL 입력
app_name="cheese", # 나중에 admin ui에서 applications에 뜬다.  
custom_headers={'Authorization': '<API token>'})  # 위에서 만든 API token Thpe "Client"
  
client.initialize_client()  
  
client.is_enabled("unleash.beta.variants")
```
- 위와 같이 연결 후 서버를 시작하면 다음과 같이 연결된 application들을 확인 할 수 있다. 
![](https://user-images.githubusercontent.com/2231510/208928977-8bd642f0-578c-4351-9915-04658a9725a6.png)


## References
- [맘편한세상에서 사용하는 피쳐 토글 서비스](https://tech.mfort.co.kr/blog/2022-11-24-feature-toggle/)
- [# Feature Toggles (aka Feature Flags)](https://martinfowler.com/articles/feature-toggles.html)