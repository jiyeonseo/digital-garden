---
title: "LaunchDarkly"
tags:
- deployment
- feature toggle
---

- [[notes/unleash]]와 거의 비슷하지만, enterprise만 제공하고 있다. 
-  stream 방식으로 polling 방식 unleash 보다 반영이 더 빠름
-   unleash와 다르게 client쪽에서 변수를 좀 더 자유롭게 사용할 수 있음. (unleash는 미리 정의해두어야함)
-   rollout 전략에서 비율 조절 뿐만 아니라 복합 속성으로 여러개 섞을 수 있음
-   [available client side도 고를수 있음.](https://docs.launchdarkly.com/home/getting-started/feature-flags?site=launchDarkly#making-flags-available-to-client-side-and-mobile-sdks)
-   [toggle scheduling도 가능](https://docs.launchdarkly.com/home/feature-workflows/scheduled-changes)