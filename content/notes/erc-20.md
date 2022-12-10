---
title: "ERC-20"
tags:
- blockchain
- ethereum
---
- [ERC-20 Token Standard](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)

- 각 토큰은 다른 토큰과 정확히 동일한 속성을 가지고 있다. ETH과 동일하게 작동하며, 1개의 토큰은 다른 토큰과 항상 같다. 

```sol
mapping(address => uint256) private _balances;
```
- 누가(adress) => 얼마(unit256) 를 가지고 있는지 저장  

## Methods 
```
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)
function balanceOf(address _owner) public view returns (uint256 balance)
function transfer(address _to, uint256 _value) public returns (bool success)
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
function approve(address _spender, uint256 _value) public returns (bool success)
function allowance(address _owner, address _spender) public view returns (uint256 remaining)

```
- `transfer` : `from` 계좌 => `to`계좌로 잔고를 바꿈 ([구현 코드](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L226-L248))

## References
- [OpenZeppelin - ERC-20 Implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)
- 