---
title: "GitHub Copoilot Chat"
tags:
- GitHub
- ChatGPT
---
GitHub은 역시 코드로 승부하나보다. VSCode Plugin으로 GitHub Copilot Chat이 베타로 오픈되었다. 업무하며 약 일주일 사용해보았고, 기존에 ChatGPT와 왔다갔다 하며 사용할때보다 훨씬 유용하고 정말 AI programming assistant 역할에 맞게 잘 수행하는 것 같다. 

## 기능 
### 1. `/explain` 
- selected code 를 자연어로 설명. 
- 익숙하지 않은 코드를 빠르게 읽어야할 때, 각 함수나 변수가 어디서 어떻게 쓰이는지 정리. 새로운 코드나 오픈소스 읽을때 아주 요긴하게 사용 가능 . 
- ChatGPT에서는 내가 코드를 다 복붙해야지만 맥락을 읽어주지만, 제가 따로 복붙을 해주지 않은 코드까지 확인하고 내가 가진 코드에서 개념을 설명해줘서 따로 이리저리 코드 여행을 다니지 않을 수 있어 좋았다. 
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/2335b511-cab3-429e-b2f6-89efcfff4aca)
- 한국어로도 가능
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/4b32a475-73ed-47d4-b28b-c0232fd793fc)


### 2. `/fix` 
- IDE에서 기본적으로 제공하는 fix는 pre defined 된 문장으로 내가 작업하고 있는 맥락과 맞지 않거나 이해하기 어려운 경우가 종종 있었음. 
- 코파일럿 챗에서는 내 코드의 맥락을 파악하여 설명해주어 더 이해하기 좋았다. 
- 코드에서 바로 대화가 가능하기도 하고. 

![](https://github.com/jiyeonseo/digital-garden/assets/2231510/a29d38ac-c48d-4048-bdad-8a2ee1534678)
- 코드 select 한 다음 `/fix`로 질문 할 수 도 있음
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/c7f83c78-61b2-46da-92c8-6d997c861d99)

### 3. `/simplify` 
- 복잡한 코드를 좀 더 간소화 버전으로 리팩토링 해주는 기능. 
- 일단 돌아가게 짠 다음, 리팩토링 혹은 코드를 간소화 하고자 할 때 이용하기 좋았음. 
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/54051bb1-9f71-4c05-af6c-ffb552656fa2)


### 4. `/tests` 
- selected code의 유닛테스트 생성. 테스트 어떻게 짤까의 힌트를 얻을 수 있다. 
- 제대로된 예제를 못찾아서 그런지 모르겠지만 크게 잘 대답해주는 것인지 모르겠다. 

### 5. Question suggestion
- 지금 chat의 맥락에서 더 고민해볼만한 질문들을 먼저 제안. 
- 대화에서 단발성 질문으로 끝나는 것이 아니라 더 깊이 언어적으로 기술적으로 함께 공부할 수 있어 좋았다. 
![twitter_Fw26Gh5aEAAQ9lM](https://github.com/jiyeonseo/digital-garden/assets/2231510/702e66a9-a9d7-40a8-9cca-8b307158d54c)
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/1469c58b-aa41-4fea-ac72-5652821598ee)


## Fun Facts 
- 좋은 코드만 짜려고 한다. 복잡한 코드 짜달라고 하니 거절한다. 
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/f8b492ac-dcf0-4fa0-aeed-edc356dba147)
번외로 ChatGPT는 잘 짜준다. 
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/a4343d60-988a-4b91-a0ba-8b29e6720b56)
 
- 코드 이외의 질문에는 대답을 안하는 편
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/165d908e-1b23-4a10-8036-4e992f7616f7)

- 물론 프롬프트를 어떻게 주느냐에 따라 다르다
![](https://github.com/jiyeonseo/digital-garden/assets/2231510/d7a16a7f-411f-4f22-8e28-186e930ca84f)

- Plugin prompt 분석한 repo
	-  https://github.com/saschaschramm/github-copilot

## References
- join wait list https://github.com/github-copilot/chat_waitlist_signup/join 