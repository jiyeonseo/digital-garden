---
title: "Poetry"
tags:
- python
- dependency manager
---

- [Python poetry](https://python-poetry.org/)

Python의 기본 패키지 매니저인 pip의 불편한점 
- 기본적으로 global 설치 : `virtualenv`와 같은 가상 환경을 따로 작업해주어야 한다. 
- lock 파일이 없어 협업시 의존성 버전 불일치가 일어날 수 있음. 

## 장점
### `poetry.lock` 

### virtualenv 
- 추가 작업없이 virtualenv를 사용할 수 있다. 

## 설치 방법 
### Linux, MacOS, Windows(WSL)
```
curl -sSL https://install.python-poetry.org | python3 -
```

### windows (powershell)
```
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
```

## 프로젝트 세팅

```sh
poetry init
```
- `pyproject.toml` 파일이 생성되며 안에 의존성이 추가된다. 

## 의존성 추가 
```sh
poetry add {package}
poetry add aioredis
```

## References
- [파이썬 의존성 관리자 Poetry 사용기](https://spoqa.github.io/2019/08/09/brand-new-python-dependency-manager-poetry.html)
