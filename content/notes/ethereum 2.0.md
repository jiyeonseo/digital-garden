---
title: "이더리움 2.0"
tags:
- ethereum
---
- PoW(작업증명) 으로 보완하지 못했던 부분들을 
- PoS(지분증명)의 시간 단위는 슬롯과 에팍

## 밸리데이터 
- 보증금 32ETH 를 예치하여야 한다.
- 역할 
	- 밸리데이터 집단을 저장 및 유지 관리 
	- Crosslinks 처리
	- 블록 합의 처리

## 슬롯 (slot)
- 시간의 가장 작은 단위 
- 12초 = 1 슬롯 (1 슬롯당 1개의 블록 형성)
- 32슬롯 = 1 에팍. 즉 384초 = 1 에팍

## 에팍 (epoch)
- 매 에팍마다 모든 밸리데이터들에게 임무가 주어짐 
- 1 에팍 = 32 슬롯 이므로, 32 밸리데이터를 선택, 각 슬롯에 배치
	- 이들이 블록 프로듀서. 블록을 만듬
	- 이 밸리데이터들은 블록 만들고 검증도 하여 "노드 위원회" 라고도 부른다.
	- 2/3 노드 위원회가 이 블록이 정확하다 승인하면 검증이 완료 됨. 
- 모든 검증이 완료되면 다음 에팍에서 새로운 슬롯에 각 밸리데이터들 배치 

## 비콘체인
- 밸리데이터들을 램덤하게 배치하고 상태 저장 관리.

## References
- [해시넷 - 슬롯](http://wiki.hash.kr/index.php/%EC%8A%AC%EB%A1%AF_(%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8))
- [# Ethereum 2.0 Phase 0 -- Beacon Chain Fork Choice](https://github.com/ethereum/annotated-spec/blob/master/phase0/fork-choice.md)