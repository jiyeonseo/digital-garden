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

## 새 프로젝트 시작하기 
```
poetry new new-poetry-repository
cd new-poetry-repository
```
들어가보면 다음과 같은 프로젝트 구조로 되어있다. ([[notes/Tree 명령어]] 를 사용)
```
new-poetry-repository/
├── README.md
├── new_poetry_repository
│   └── __init__.py
├── pyproject.toml
└── tests
    └── __init__.py
```

만약, 패키지 이름을 바꾸려면 아래와 같이 `name`을 따로 줄 수도 있다.
```
poetry new new-poetry-repository --name cheese-poetry
```

```
new-poetry-repository
├── README.md
├── cheese_poetry
│   └── __init__.py
├── pyproject.toml
└── tests
    └── __init__.py
```

## 이미 있는 프로젝트 내에 세팅하기

```sh
poetry init
```
- `pyproject.toml` 파일이 생성되며 안에 의존성이 추가된다. 

## 의존성 추가 
```sh
poetry add {package}
poetry add aioredis
```

## `pyproject.toml`

```toml
[tool.poetry]
name = "cheese-poetry"
version = "0.1.0"
description = ""
authors = ["cheese <seojeee@gmail.com>"]
readme = "README.md"
packages = [{include = "cheese_poetry"}]

[tool.poetry.dependencies]
python = "^3.10"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### `poetry check`
 - `pyproject.toml` 파일 validation

### `poetry install`
 - `pyproject.toml` 의 의존성 로컬에 설치하기 
```sh
poetry install --no-dev # dev-dependencies 빼고 
```
### `poetry update`
- 의존성 모두 업데이트 하기 
- `poetry.lock`을 삭제하고 새로 `poetry install` 하는 것과 동일한 동작

## poetry.lock
- 설치된 패키지의 정확한 버전을 명시하여 어떠한 특정 버전이 이 프로젝트에서 사용되고 있는지 lock 
- 여러 사람과 협업시 동일한 버전의 의존성을 가질 수 있게 됨
- 만약 `poetry.lock` 이 없다면 `pyproject.toml` 에 명시된 내용에서 가장 최신 버전을 받게 된다.

## Virtual Environment
```sh
# 설정 가능한 python 환경들 확인하기 
poetry env list

# venv로 사용할 python 환경
poetry env use python3

# venv 사용하여 python 사용하기 
poetry run python ./main.py
```
지금 사용중인 virtual env 위치 확인하기 
```sh
poetry show -v
```


## References
- [파이썬 의존성 관리자 Poetry 사용기](https://spoqa.github.io/2019/08/09/brand-new-python-dependency-manager-poetry.html)
- [Dependency management with Python Poetry](https://realpython.com/dependency-management-python-poetry/)