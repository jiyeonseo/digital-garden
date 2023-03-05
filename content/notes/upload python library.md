---
title: "Python Project package PyPi에 올리기"
tags:
- python
- PyPI
---

## pip 설치
- Unix/mac OS
```sh
python3 -m pip install --upgrade pip
```
- windows
```
py -m pip install --upgrade pip
```

## Project 생성
간단한 프로젝트 `packaging_tutorial` 다음과 같은 구성으로 준비한다. 
```
packaging_tutorial/ 
└── src/
    └── example_package_cheese/
        ├── __init__.py
        └── example.py
```
- `example_package_cheese` : project 명과 동일하게 해주는 것이 좋다. 설정도 더 간편하고 패키지사용자 입장에서도 구분하기 좋다. 
- `__init__.py` : 비어있는 파일. directory 자체를 package로 import 하기 위해 필요.
- `example.py` : 실제 로직이 담길 파일. 아래와 같이 실제 로직을 추가해준다. 
```pyhon
def add_one(number):
    return number + 1
```

## Package Files 생성

```
packaging_tutorial/
├── LICENSE
├── pyproject.toml
├── README.md
├── src/
│   └── example_package_YOUR_USERNAME_HERE/
│       ├── __init__.py
│       └── example.py
└── tests/
```
- `pyproject.toml` : 프로젝트 빌드 시스템 요구 사항과 정보를 담고 있는 파일 
	- 빌드툴로는 [Hatchling](https://packaging.python.org/en/latest/key_projects/#hatch), [setuptools](https://packaging.python.org/en/latest/key_projects/#setuptools), [Filt](https://packaging.python.org/en/latest/key_projects/#flit). [PDM](https://packaging.python.org/en/latest/key_projects/#pdm) 등이 있으며 여기서는 Hatchling으로 진행한다. 
```
# Hatchlling
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# setuptools
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

# filt
[build-system]
requires = ["flit_core>=3.4"]
build-backend = "flit_core.buildapi"
```
- `requires` : 패키지를 빌드하는데 필요한 패키지 목록. 
	- pip 와 같은 빌드 프론트엔드들에서 자동으로 격리된 virtual enviroment에서 임시로 설치하여 빌드에 사용되기 때문에 따로 설치할 필요는 없다. 
- `build-backend` : 빌드 프론트엔드에서 빌드할때 사용하는 Python 객체명

###  `pyproject.toml` 에 metadata 작성하기 
```toml
[project]
name = "example_package_YOUR_USERNAME_HERE"
version = "0.0.1"
authors = [
  { name="Example Author", email="author@example.com" },
]
description = "A small example package"
readme = "README.md"
requires-python = ">=3.7"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]

[project.urls]
"Homepage" = "https://github.com/pypa/sampleproject"
"Bug Tracker" = "https://github.com/pypa/sampleproject/issues"
```
- `name` : 패키지의 이름. PyPI에 이미 올라와 있는 이름과 겹칠 수 없다. 
- `version` : 패키지의 버전.  [version spec](https://packaging.python.org/en/latest/specifications/version-specifiers/#version-specifiers)
- `authors` : 작성자. `maintainers`도 동일한 모양으로 작성할 수 있다.
- `readme` :  패키지 readme 파일 path
- `requires-python` : 필요 python 버전 명시
- `classifiers` : 추가적인 메타데이터. 호환 python 버전, 라이센스, OS 등. [PyPi classifier 목록](https://pypi.org/classifiers/)
- `[project]` 에 사용할 수 있는 더 많은 메타데이터들은 [project metadata specification](https://packaging.python.org/en/latest/specifications/declaring-project-metadata/#declaring-project-metadata) 에서 확인 할 수 있다. 
- 이외에도 `keyword` 를 통해 사용자들이 더 찾기 쉽게 키워드를 등록할 수도 있고, `dependencies`를 이용하여 패키지에 필요한 dependecies들도 관리 할 수 있다. 

## 패키지 빌드하기 
pip build를 위하여 PyPA build 설치 
```
# Unix/mac OS
py -m pip install --upgrade build

# Windows
python3 -m pip install --upgrade build
```

위에 작성한 `pyproject.toml` 파일 위치에서 아래 커맨드로 빌드하기 
```
# Unix/mac OS
python3 -m build

# Windows
py -m build
```

빌드하고 나면 아래와 같이 `/dist` directory가 생성된다. 
```
dist/
├── example_package_YOUR_USERNAME_HERE-0.0.1-py3-none-any.whl
└── example_package_YOUR_USERNAME_HERE-0.0.1.tar.gz
```
- `.tar.gz` : [source distribution](https://packaging.python.org/en/latest/glossary/#term-Source-Distribution-or-sdist)
- `.whl` : [built distribution](https://packaging.python.org/en/latest/glossary/#term-Built-Distribution) 
- 최신 pip는 `.whl` 로 우선적으로 설치되지만, 호환이 안되는 경우 `.tar.gz`으로 

## 패키지 업로드하기 
- 테스트 업로드를 하기 위해 TestPyPI 계정 만들기 
	- https://test.pypi.org/account/register/ (email verification 반드시 필요)
	- 더 자세한 사항은 [Using TestPyPI](https://packaging.python.org/en/latest/guides/using-testpypi/) 를 참고
	- 업로드를 위해서 [API Token](https://test.pypi.org/help/#apitoken) 을 사용하는데 API token은  [TestPyPI account](https://test.pypi.org/manage/account/#api-tokens) 에서 생성할 수 있다. 

업로드 하기 위하여 [twine](https://packaging.python.org/en/latest/key_projects/#twine) 을 먼저 설치
```
# Unix/macOS
python3 -m pip install --upgrade twine

# Windows
py -m pip install --upgrade twine
```

twine 이용하여 아래와 같이 업로드 할 수 있다. 
```
# Unix/macOS
python3 -m twine upload --repository testpypi dist/*

# Windows
py -m twine upload --repository testpypi dist/*
```

이 과정에서 username과 password를 입력하도록 되어있는데, 
- username :  `__token__` 
- password : 위에서 발급받은 API Token(`pypi-blahblah`)을 입력 
위 업로드가 성공하면 `https://test.pypi.org/project/example_package_YOUR_USERNAME_HERE`. 에서 올라간 패키지를 확인할 수 있다. 
> 예시 : https://test.pypi.org/project/cheese-package/0.1.0/

## 패키지 설치하기 
```
# Unix/macOS
python3 -m pip install --index-url https://test.pypi.org/simple/ --no-deps example-package-YOUR-USERNAME-HERE

# Windows
py -m pip install --index-url https://test.pypi.org/simple/ --no-deps example-package-YOUR-USERNAME-HERE
```
- `--index--url`
- `--no-deps`


## References 
- [Packaging Python Projects](https://packaging.python.org/en/latest/tutorials/packaging-projects/)
- [circleci blog - Publishing a Python package](https://circleci.com/blog/publishing-a-python-package/?utm_source=google&utm_medium=sem&utm_campaign=sem-google-dg--japac-en-dsa-tROAS-auth-brand&utm_term=g_-_c__dsa_&utm_content=&gclid=Cj0KCQiA9YugBhCZARIsAACXxeJChlMZzUeuiklpIknQj03iGqmlmwy4dj2MJIZ0xvahOHsyn1TB0wUaApS3EALw_wcB)