---
title: "Dockerfile"
tags:
- docker
---

- Docker 컨테이너 이미지를 빌드 내용을 담고 있는 텍스트 파일 

## 구성 
- 베이스 이미지 설정 : `Ubuntu`, `CentOS` 등 기본 운영 체제 및 환경 설정 
- 패키지 및 라이브러리 설치  : 컨테이너 구동을 위한 패키지나 라이브러리, 어플리케이션 등을 설치. `RUN` 명령어 사용 
- 소스 코드 및 파일 복사  : `COPY` 혹은 `ADD`명령어 실행 
- 환경 변수 설정 : Port를 열거나 실행할 명령어 등 실행 

## 간단한 예시 

```dockerfile
# 베이스 이미지를 선택
FROM ubuntu:20.04

# 필요한 패키지 설치
RUN apt-get update && apt-get install -y nginx

# 로컬 코드 파일 복사 (dockerfile이 있는 path)
COPY index.html /var/www/html/

# 포트 80 열기
EXPOSE 80

# 웹 서버 실행
CMD ["nginx", "-g", "daemon off;"]

```

## Docker 이미지 생성 방법 
```
(dockerfile이 있는 path로 이동 후)
$ docker build 
$ docker build -t {tag} .
```

## Tips

### `.dockerignore` 파일

- 복사하지 않을 파일들을 미리 정의하여 `ADD`나 `COPY` 사용시에 이미지에 추가 되는 것을 방지할 수 있다. 
- 용량이 큰데 필요 없거나 민감한 정보 파일 같은 경우 사용 할 수 있다. 
- [`.dockerignore` 문서](https://docs.docker.com/engine/reference/builder/#dockerignore-file)

```gitignore
# comment
*/temp*
*/*/temp*
temp?
```

- 매칭 룰은 Go의 [filepath.Match](https://pkg.go.dev/path/filepath#Match) 룰을 따른다. 

### `ADD` 보다는 `COPY`
- 두 명령어 모두 이미지에 파일과 디렉토리를 복사하는 것이지만 차이가 있다. 
- `ADD`는 다음과 같은 기능들을 더 가지고 있다
	- 압축 파일 추출 : `.tar`, `.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz2` 와 같은 압축파일을 자동으로 푸는 기능
	- 캐시 무효화 : `ADD <src>` 파일 내용이 변경되면 캐시를 무효화 시킨다. 
	- 권한 유지 : `ADD` 사용하면 파일/디렉토리의 권한이 유지. `COPY`는 권한이 기본적으로 `755` 설정된다. 
	- 파일 URL 지원 : URL 변수를 지원하여, 웹사이트나 Git 리파지토리와 같은 원격 위치의 파일을 복사하는데 사용할 수 있다. 
- 하지만, 공식문서에서는 예측할 수 없는 동작이 발생하지 않는 `COPY`를 쓰는 것을 추천하고 있다.
- [`ADD` or `COPY` 공식문서](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy)

### Multi-stage builds

### Multi-layers and caching

### `ONBUILD`

### `USER` 변경하기 

### 컨테이너 health 체크하기 

### `SHELL` 사용하기

### 메타데이터 생성 

### Dockerfile when built, is only a tar

## References 
- [10 Secrets to Improve Your Dockerfile](https://aws.plainenglish.io/10-secrets-to-improve-your-dockerfile-40ac54aa5bf2)
- 