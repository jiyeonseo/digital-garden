---
title: "ABI"
tags:
- blockchain
- ethereum
---

- Contract **A**pplication **B**inary **I**nterface
- Smart Contract 함수와 파라미터에 대한 Interface 정의 
- 가지고 있는 정보
	- 함수에 대한 **정보** 
	- 함수에서 사용하는 **인수**
![](https://static.packt-cdn.com/products/9781789954111/graphics/assets/fe0f2ffc-2f3c-4615-9cb5-43c8e036239b.png)
- 예시 : https://etherscan.io/address/0xb59f67a8bff5d8cd03f6ac17265c550ed8f33907#code
```json
{
   "anonymous":false,
   "inputs":[
      {
         "indexed":true,
         "name":"from",
         "type":"address"
      },
      {
         "indexed":true,
         "name":"to",
         "type":"address"
      },
      {
         "indexed":false,
         "name":"value",
         "type":"uint256"
      }
   ],
   "name":"Transfer",
   "type":"event"
},
{
   "anonymous":false,
   "inputs":[
      {
         "indexed":true,
         "name":"old",
         "type":"address"
      },
      {
         "indexed":true,
         "name":"current",
         "type":"address"
      }
   ],
   "name":"NewOwner",
   "type":"event"
}
```
- `anonymous` : 이 method가 public인지 아닌지 (default: `false`. public 이다.)
- `type` : 어떤 데이터 타입인지
	- `event`
	- `inputs` 안에는 input type
- `name` : item 혹은 파라미터의 이름 
- `indexed` 
	- `true` : `topics` 에 저장된다. 
	- `false` : `data` 필드에 들어간다. 


## References
- [Understanding Logs: Deep Dive into eth_getLogs](https://docs.alchemy.com/docs/deep-dive-into-eth_getlogs)
- [Understanding event logs on the Ethereum blockchain](https://medium.com/mycrypto/understanding-event-logs-on-the-ethereum-blockchain-f4ae7ba50378)