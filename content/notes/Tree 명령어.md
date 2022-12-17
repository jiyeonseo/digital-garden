---
title: "Tree 명령어"
tags:
- MacOS
---
- 폴더 구조를 시각적으로 보기 쉽도록 트리 형태로 보여주는 명령어

## Install

### MacOS (Homebrew)
```
brew install tree
```

### Linux (RHEL / CentOS / Fedora / Rocky / Alma Linux)
```
yum install tree  
## CentOS/RHEL 8.x and Fedora user try the dnf command ##  
dnf install tree
```

### Linux (Debian / Mint / Ubuntu Linux)
```
sudo apt-get install tree
```

## How to use
```
tree .

➜  new-poetry-repository tree
.
├── README.md
├── new_poetry_repository
│   └── __init__.py
├── pyproject.toml
└── tests
    └── __init__.py

```

## References
- [Linux see directory tree structure using tree command](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/)