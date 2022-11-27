---
title: "Ethereum"
tags:
- blockchain
- ethereum
---

## Ethereum
- 컴퓨터가 내장된 블록체인.
- 비트코인과 다른 점 : 암호화폐로서의 기능 뿐만 아니라 프로그래밍이 가능. 네트워크에서 분산형 애플리케이션을 구축 및 배포할 수 있음.
- 중앙 주체의 통제 없이 앱과 조직을 구축하고, 자산을 보유하고, 거래하고 소통할 수 있음

## EVM
- Ethereum Virtual Machine
- 글로벌 가상 컴퓨터, 모든 참여자들이 코드 실행을 요청 할 수 있고, 코드 실행은 EVM 상태를 변경시킴
- [https://ethereum.org/ko/developers/docs/evm](https://ethereum.org/ko/developers/docs/evm)

## Ether
- ETH : native cryptocurrency of Ethereum
- 화폐로서 사용되는 암호화폐
- 거래 요청 확인 및 실행한 컴퓨터에 대한 보상으로 지급됨

## Ethereum Classic
- 오리지널 이더리움 네트워크 해킹을 해결하기 위해 하드포크 하기 전의 오픈소스 블록체인
- ETC : 이더리움 클래식의 고유 통화

## Smart contracts
- 이더리움 블록체인 상에서 실행되는 프로그램
- 이더리움 블록체인 상의 특정 주소에 있는 코드(function)과 데이터(state)의 모음

### Smart contract 동작 방식
- 네트워크 상에 미리 결정된 조건이 충족될 때 동작 실행

## Minting Ether
- 이더리움 원장(Ethereum ledger)에 새로운 이더리움을 발행하는 것
- underlying Ethereum 만이 새로운 ether를 만들 수 있다.

## Burning Ether
- 이더리움 원장(Ethereum ledger)에서 이더리움을 삭제하는 것

## Denominations of ether
- 이더 종파
- 이더리움에서 적은 금액으로 거래가 자주 일어나, 이를 용이하게 하기 위해 만든 단위
- "Wei"와 "Gwei"가 가장 유명하다.
- [https://ethereum.org/ko/developers/docs/intro-to-ether/#denominations](https://ethereum.org/ko/developers/docs/intro-to-ether/#denominations)

### Wei
- 이더(ether)의 가장 작은 단위
- 1 wei = 10^-18 ether = 0.000000000000000001 ether
- 1 ether = 10^18 wei = 1,000,000,000,000,000,000 wei
- 주로 기술적 구현단에서 자주 사용됨.
- 비트코인 탄생에 큰 영향을 준 B-Money를 고안한 인물인 [Wei Dai](https://en.wikipedia.org/wiki/Wei_Dai)의 이름을 따서 만들어졌다.

### Gwei
- giga-wei. 1,000,000,000 wei
- Gas 이야기 할 때 많이 사용됨.

## Transferring ether
- 각 트랜잭션에는 `value` 필드 : 전송할 ether의 양(wei 단위로)
- sender -> recipient 주소로 옮겨짐
- recipient 주소가 smart contract라면 smart contract 실행에 대한 gas fee 지불하는데 새용될 수도 있다.
- [https://ethereum.org/ko/developers/docs/intro-to-ether/#transferring-ether](https://ethereum.org/ko/developers/docs/intro-to-ether/#transferring-ether)

  
## Ethereum accounts
- 이더리움에서 거래할 수 있는 ether 잔고를 가지고 있는 entity
- account은 사용자가 사용하거나 smart contract를 배포할 수 있다.

## Types of Ethereum accounts
- 1) Externally-owned account (EOA) : priveate key를 가지고 있는 계정
- 2) Contract account : 네트워크에 배포된 smart contract.

### 1) Externally-owned account (EOA)
- 만드는데 비용이 들지 않는다.
- 트랜잭션을 일으킬 수 있다.
- EOA 사이에서 ETH나 토큰을 전송할 수 있다.
- public key와 private key 암호화된 키 쌍을 가지고 있다.

### 2) Contract account
- 생성시 비용이 든다. 네트워크 스토리지를 사용하기 때문에
- 트랜잭션 수신하는 응답으로만 트랜잭션을 보낼 수 있다. (스스로 트랜잭션을 보낼 수 없다.)
- 외부에서 contract account로의 전송을 통해 여러 가지 작업할 수 있는 코드를 트리거링 할 수 있다. ex. 토큰 보내기 혹은 새로운 contract 만들기 등
- private 키가 없다. 대신 code 내 로직에 의해 컨트롤 된다.


## Ethereum accounts have four fields
![](https://ethereum.org/static/19443ab40f108c985fb95b07bac29bcb/302a4/accounts.png)
 - [AN ACCOUNT EXAMINED](https://ethereum.org/ko/developers/docs/accounts/#an-account-examined)

### nonce
- 각 account에서 보낸 트랜잭션의 수. 트랜잭션 카운터
- 한 트랜잭션을 한번만 처리할 수 있게 됨. replay attack 방지.
- account 에서 생성된 contract 갯수.

### balance
- 주소가 소유한 wei의 수.

### codeHash
- EVM(이더리움 가상머신) 내 account 코드.
- 변하지 않는 값. (다른 필드들은 변함)
- contract account의 경우, contract의 코드를 가리킨다.
- account가 메세지를 받게되면 코드 실행
- 코드 조각은 나중에 검색할 수 있도록 해당 해시 아래 상태 데이터베이스(state database)에 포함됨
- EOA의 경우, 빈 string 해시 값.

### storageRoot
- storage hash
- account의 storage content의 해시 값.
- default는 비어 있음.

## References
- Ethereum official website : [https://ethereum.org/ko/what-is-ethereum/](https://ethereum.org/ko/what-is-ethereum/)
- [Learn Ethereum Blockchain daily and Keep the Knowledge Awake :)](https://medium.com/coinsbench/learn-ethereum-blockchain-daily-and-keep-the-knowledge-awake-day-1-6d482ae67ac7)
- [Account Abstraction & ERC 4337](https://medium.com/decipher-media/account-abstraction-erc-4337-2b8dff6b0a34)