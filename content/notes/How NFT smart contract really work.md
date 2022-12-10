---
title: "How NFT smart contract really work"
tags:
- blockchain
- ethereum
---

> Ethereum NFT 기준

## OpenSea 

- 예제 [Doodle #1815](https://opensea.io/assets/ethereum/0x8a90cab2b38dba80c64b7734e58ee1db38b8992e/1815)
![](https://user-images.githubusercontent.com/2231510/204299525-29388efa-cc3f-4fed-bd2b-9acc83e8d2a5.png")
![](https://user-images.githubusercontent.com/2231510/204299525-29388efa-cc3f-4fed-bd2b-9acc83e8d2a5.png)

- OpenSea 주소에서 `chain`, `contract address`, `token` 확인할 수 있다.
- `https://opensea.io/assets/{chain}/{contract address}/{token id}`

![](https://user-images.githubusercontent.com/2231510/204303462-86a6c32d-033c-4052-ac52-c559d0a2c944.png)
- Details 에서도 동일하게 NFT에 대해 알 수 있다.
- Contract Address 링크 : NFT Collection contract [[notes/Etherscan]] 페이지로 연결
- Token ID 링크 : NFT Token의 metadata 

## Etherscan
- [예제 Contract 0x8a90CAb2b38dba80c64b7734e58Ee1dB38B8992e](https://etherscan.io/address/0x8a90cab2b38dba80c64b7734e58ee1db38b8992e)

### "Contract" tab
- Code
	- Contract Source Code : solidity. contract code 내용
	- Contract ABI 
	- Contract Creation Code
		- ByteCode
		- Opcodes
- Read Contract : 해당 스마트 컨트렉트에 대해 READ 할수 있는 function들 
	- 예를 들어, `onwnerOf`  function에 `tokenId`를 넣으면 해당 `tokenId`의 NFT를 가진 owner query 가능
	  ![](https://user-images.githubusercontent.com/2231510/204311713-6f1ea517-9144-43bc-9add-3e39f0ce9386.png)
	- `tokenURI` : NFT metadata link 
	  ![](https://user-images.githubusercontent.com/2231510/204314326-005287a8-b0f2-43d6-a94a-dd5b8c4bac92.png)
	  - [[notes/IPFS]] 주소
	  - `https://ipfs.io/ipfs/{뒤쪽 주소}` 
		  - 앞부분은 바뀌지 않고 맨 뒤 token ID만 변경된다. 
	  - [https://ipfs.io/ipfs/QmPMc4tcBsMqLRuCQtPmPe84bpSjrC3Ky7t3JWuHXYB4aS/13](https://ipfs.io/ipfs/QmPMc4tcBsMqLRuCQtPmPe84bpSjrC3Ky7t3JWuHXYB4aS/13)
	  - 이렇게 다른 곳에 저장되어있기 때문에 **owner**가 원하면 metada를 바꿀 수 있다.
	  - owner를 만약 resign(이 역시도 function) 한다면 owner가 없게 되고 해당 NFT는 영원히 바꿀 수 없게 된다. 
	- `balanceOf` : owner 주소가 해당 contract NFT 를 몇개 가졌는지 
	  ![](https://user-images.githubusercontent.com/2231510/204318782-5f27a4cd-4f0d-42ff-92cc-04e2d2cdf3d7.png)
	- `totalSupply` : 이 contract의 최대 발행갯수 
	  ![](https://user-images.githubusercontent.com/2231510/204319392-9b9c2e31-1b97-43f7-8a67-ee509a4594a2.png)
	  
- Write code 
	- 지갑과 연결하여 code를 실행시킬 수 있다. 
	- `setBaseURI` : Metadata URL 세팅하기 
	  ![](https://user-images.githubusercontent.com/2231510/204317487-52539b7e-2fe6-444f-b450-f93f8cd83604.png)
	  (내가 `owner`가 아니기 때문에 denied 됨)
	  ![](https://user-images.githubusercontent.com/2231510/204317791-d5d95e9d-1074-453d-b198-e8f78f78c8d6.png)
	  (코드 보면 `onlyOwner` contract owner 만 가능하게 되어있음 )
	  나중에 metadata가 호오오옥시나 바뀌게 되면 이 `setBaseURI`로 변경할 수 있음. 
	  - `withdraw` : `onlyOwner`
	    해당 contract의 balance를 해당 function call 한 사람에게 transfer 한다. (누구든 부를 수 있긴 하지만 `onlyOwner`에서 막히니 owner만이 balance를 가져갈 수 있다.)
	    ![](https://user-images.githubusercontent.com/2231510/204320002-9f55dc99-5744-4f15-ac68-8cc8d5480336.png)
	    
## NFT Staking
- [예시 Wizards & Dragons Game (WnD)](https://opensea.io/collection/wizards-dragons-game-v2)
- [Contract Etherscan](https://etherscan.io/address/0x999e88075692bcee3dbc07e7e64cd32f39a1d3ab#readContract)
- `Contract` tab > `Read Contract` > `tower` 다른 contract 주소가 있음
  ![](https://user-images.githubusercontent.com/2231510/204322362-d41719e9-015d-403f-91e0-8bc9986dfd42.png)
  staking contract 
  ![](https://user-images.githubusercontent.com/2231510/204323453-a5c3738a-c8d6-45cd-a02b-40523d298f07.png)
  `transferFrom` : [[notes/ERC-20]] standard function
	  - `tokenOwner` 로 부터 이 `address` 에게 NFT를 보내겠다. 
- unstaking
  ![](https://user-images.githubusercontent.com/2231510/204324932-81b03754-0647-4039-9982-b511ff4bca45.png)
  "claim", "unstaking" function을 보면 대부분 여기에 reward에 대한 코드가 있다. 


## References
- [HOW NFT SMART CONTRACTS REALLY WORK - Can metadata be changed? How staking works?](https://www.youtube.com/watch?v=Wu436_IwWmo)