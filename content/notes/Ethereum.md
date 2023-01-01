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
- [[notes/Ethereum Virtual Machine]]
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

## Contract address
- 42자로 구성된 16진수 주소
	- ex. `0x06012c8cf97bead5deae237070f9587f8e7a266d`
- Contract가 이더리움 블록체인에 배포될 때 부여된다.
- 만든 사람의 주소와 해당 주소에서 보낸 트랜잭션의 수([nonce](#nonce))를 통해 만들어진다.

## Validator Keys
- 작업 증명(proof-of-work)에서 지분 증명(proof-of-stake)로 변경되며 필요해짐
- BLS(Boneh-Lyn-Shacham) 키는 검증인(validator)를 식별하는데 사용 
- BLS키는 효율적으로 집계하여 네트워크 합의 도달하는데 필요한 대역폭을 줄일 수 있다. 
- private key + public key

## Wallets
- account != wallet
- account와 인터렉션 할 수 있는 interface 이자 application

## Querying Ether
- [[notes/Etherscan]]에서 주소 검색하여 잔액을 확인할 수 있다. 

## DApp
- decentralized application
- decentralized network 상 위에 smart contract와 front end 인터페이스로 만들어진 어플리케이션. 
	- smart contract는 접근 가능하며 투명하게 공개된다. 
- 장점
	- Zero downtime
		- 블록체인 네트워크 상에서 운영되기 때문에 
	- privacy
	- 검열이 따로 없음
	- 데이터 무결성
		- 블록체인에 데이터가 저장되기 때문에 불변이며 위조할 수 없다. 
	- 검증 가능한 동작 
		- 별도 중앙 기관 없이 따로 신뢰할 것 없이 사용 가능하다. 
		- 현실 세계에서는  은행 시스템 사용시 금융 기관을 믿고 사용하지만 DApp에서는 그럴 필요가 없다.
- 단점
	- 운영이 어렵다.
		- 블록체인에 올라가고나면 수정이 어렵다. 
	- 퍼포먼스 이슈 
		- 이더리움이 지향하는 보안, 무결성, 투명성, 신뢰성 수준 달성을 위해 모든 노드가 모든 트랜잭션을 실행 및 저장. 증명 합의에도 시간이 걸리게 된다. 
	- 네트워크 정체
		- 하나의 DApp이 너무 많은 리소스를 사용하게 되면 전체 네트워크가 정체된다. 현재 네트워크는 초당 10~15개의 트랜잭션만 처리 가능하며, 이보다 더 많은 트랜잭션이 생기면 풀이 빠르게 증가 할 수 있다. 
	- 사용성
		- 사용자 역시 블록체인의 보안 방식을 설정해야함으로 진입 장벽이 높을 수 있다.
	- 중앙 집중화 
		- 사용자 친화적으로 하려면 전통적인 중앙 집중화된 서비스로 만들게 되는데, 이렇게 되면 블록체인의 장점들을 더 이상 사용할 수 없게 된다. 

## Transaction
- account의 암호화된 명령
- 예를 들어, account => account ETH 전송
- [EVM](#EVM) 상태를 변경하는 트랜잭션은 전체 네트워크로 브로드캐스트 되어야 한다.
	- **recipient** : 수신 주소 
		- EOA : 전송 받는 사람
		- Contact account : 이 트랜잭션은 contract code를 실행하는 트랜잭션임. 
	- **signature** : 보낸 사람 식별자 
		- 보낸 사람의 private key가 트랜잭션에 싸인하고 보낸 사람이 이 트랜잭션을 승인 했다고 확인 할 때 생성됨. 
	- **nonce** : 계정의 트랜잭션 번호. 순차적으로 증가하는 카운터
	- **value** : 전송하는 ETH 양 ([WEI](#wei))
	- **data** : 임의 데이터 (optional)
	- **gasLimit** : 트랜잭션에서 사용할 수 있는 최대 가스 단위의 양. 계산 단계에서 나오는 가스 양
	- **maxPriorityFeePerGas** : validator에게 보낼 최대 가스량
	- **maxFeePerGas** : 거래에 대해 지불할 수 있는 최대 가스량 ( baseFeePerGas 와 maxPriorityFeePerGas 포함)

## Gas
- 이더리움 네트워크에서 특정 작업을 실행하는데 필요한 계산 노력의 양을 측정하는 단위 
- 거래를 실행하기 위해 계산 자원 필요 => 각 거래는 수수료가 필요.
- 이 거래를 성공적으로 수행하기 위한 필요한 수수료 
- `gasLimit` 와 `maxPriorityFeePerGas` 를 이용하여 validator에게 줄 최대 거래 수수료를 결정한다. 

### 거래 수수료 계산 방법 
- 이더리움 네트워크 거래 수수료 계산 방식은 [[notes/London upgrade]] 이후로 변경됨.
- 기존 동작 방식 (런던 업그레이드 전)
	- `gasLimit` : 21,000 unit, gas price : 200 gwei 인 경우 => Gas units * price = 21,000 * 200 = 4,2000,000 gwei (0.0042 ETH)
- 이후 동작 방식 (런던 업그레이드 이후)
	- gasLimit` : 21,000 unit`, **base fee** 10 gwei, tip은 2 gwei 
	  => 21,000 ( 10 + 2) = 252,000 gwei (0.000252ETH)
	- validator는 0.000042 ETH 팁을 받고, base fee인 0.00021 ETH는 burn 된다.
	- `maxFeePerGas` 설정할 수 있다.
		- refund = 최대 수수료 - ( base fee + priority fee )
	- 이를 통해 기본료 이상의 과도한 지불을 막는다.

## Block
- 트랜잭션을 담고 있다. 
- 즉, 트랜잭션은 기록이고 블록은 장부다. 
- hash는 블록 데이터에서 암호화되어 이 링크로 블록들을 연결 
	- 한번 연결된 hash는 모든 이용자가 알고 있기 때문에 쉽게 변조할 수 없다. 

### 왜 block이 필요한가
- 이더리움 네트워크에 잇는 모든 참가자가 동기화 상태를 유지하고 트랜잭션의 정확한 기록을 동의
- 밸리데이터(validator)들은 랜덤으로 트랜잭션을 블록에 담을 수 있는 권한을 받고 그 트랜잭션을 블록에 담아 또 다른 밸리데이터들에게 공유 

### block 작동 방식 
- 트랜잭션 기록 보존을 위해 블록은 반드시 정렬된다
- 블록이 밸리데이터에 의해 합쳐지고 나면, 네트워크에 모두 전파된다.
- 블록이 합쳐지고 나면 새로운 벨리데이터를 선택하고 다음 블록을 만든다.
- 현재 이더리움은 proof-of-stake(POS) 프로토콜로 명시되어있다. 

### Proof-of-stake Protocol 
- 지분 증명 
- 밸리데이터가 되기 위해서는 담보로 32ETH 지불해야한다. 
- 부정행위가 걸릴 경우, 이 담보에서 사라지게 됨으로 네트워크 보호용으로 사용된다.
- 모든 [[notes/ethereum 2.0#슬롯 (slot)]]마다 [[notes/Block Proposer]]로 부터 밸리데이터가 무작위로 선택
- 밸리데이터는 트랜잭션을 묶고, 실행, 새로운 상태(state)를 결정 후 다른 밸리데이터들에게 전달 
	- 다른 밸리데이터들 : 변경 사항들을 실행해보고, 유효하다고 판단하면 해당 블록을 자체 데이터베이스에 추가 
- 동일 슬롯에 대해 두개의 충돌 블록이 있을 경우 [fork-choice algorithm](https://github.com/ethereum/annotated-spec/blob/master/phase0/fork-choice.md) 으로 가장 많이 연결된 ETH가 지원하는 블록 선택

### What's in a block
![](https://user-images.githubusercontent.com/2231510/205472611-01a88b46-e4f4-4d97-8907-1d2f095761b3.png)
- `slot` : 이 블록이 속한 슬롯 
- `proposer_index` : 블록 제안한 밸리데이터의 ID
- `parent_root` : 이전 블록 해시
- `state_root` : 상태 오브젝트의 루트 해시
- `body`
	- `randao_reveal` : 다음 [[notes/Block Proposer]]을 위해 사용될 값 
	- `eth1_data` : 담보 컨트렉트 정보
	- `graffiti` : 블록 태그를 위한 임의 데이터
	- `proposer_slashings` :  잘라낼 밸리데이터 목록
	- `attester_slashings` : 잘라낼 밸리데이터 목록
	- `attestations` : 현재 블록을 지원하는 증명 목록
	- `deposits` : 담보 컨트랙트를 위한 새로운 담보 리스트 
	- `voluntary_exits` : 네트워크 내 밸리데이터 목록
	- `sync_aggregate` : light client에 전달하는 밸리데이터의 subset
	- `execution_payload` : 실행 client로 부터의 트랜잭션

### Block Time
- 블록을 분리하는 시간
- 이더리움에서 시간은 슬롯([[notes/ethereum 2.0#슬롯 (slot)]] = 12초
- 모든 밸리데이터가 온라인 상태고 완전히 동작한다고 가정했을 때, 모든 슬롯에 블록이 있다. -> 즉, 블록 시간은 **12초** 
- 하지만, 밸리데이터가 오프라인일수도 있어서, 슬롯이 빌 수도 있음. 

### Block Size
- 각 블록의 목표 사이즈는 1500만 가스.
- 하지만 네트워크 수요에 따라 증가, 감소가 가능하여 최대 3000만 가스(목표의 2배)까지 늘어날 수 있다.
- 블록 내 모든 트랜잭션에서 지출되는 가스피의 총량은 블록 가스 한도보다 작아야 한다.
	- 블록이 임의로 커져버리지 않게 하기위한 장치
	- 블록이 너무 커지면 전체 노드의 리소스와 속도 요구사항으로 인해 네트워크를 따라갈 수 없게 될 수 있음

### Base fee
- 모든 블록에는 base fee가 있어 예비 가격 역할을 함
- 블록에 들어가려면 이 base fee 보다는 커야 함
- 각각의 블록마다 독립적으로 계산됨. 
- 이전 블록에 의해 결정됨으로 사용자는 fee를 예측할 수 있음
- 블록이 채굴될때마다 연소(burned)된다. 

### Priority fee (tips)
- 

### Max fee



## References
- Ethereum official website : [https://ethereum.org/ko/what-is-ethereum/](https://ethereum.org/ko/what-is-ethereum/)
- [Learn Ethereum Blockchain daily and Keep the Knowledge Awake :)](https://medium.com/coinsbench/learn-ethereum-blockchain-daily-and-keep-the-knowledge-awake-day-1-6d482ae67ac7)
- [Account Abstraction & ERC 4337](https://medium.com/decipher-media/account-abstraction-erc-4337-2b8dff6b0a34)