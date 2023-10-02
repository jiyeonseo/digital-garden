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

## 명령어

### `RUN`
- `RUN <command>` : shell 형식으로 실행 
- `RUN ["executable", "param1", "param2"]` 
- 현재 이미지 위에 새 Layer에서 모든 명령을 실행하고 결과 커밋 -> 커밋된 결과는 다음 단계에서 계속 사용됨 


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
- 단일 dockerfile에서도 여러개의 `FROM`문을 이용하여 여러 스테이지의 빌드를 생성 할 수 있다. 
- 즉, 한 환경에서 어플리케이션 빌드 후, 그 결과물을 다른 이미지로 복사하여 사용 가능 
- 어플리케이션 실행에 필요하지 않은 **중간 빌드 결과물을 제외할 수 있어, 최종 이미지 크기를 줄이는데 유용**

```dockerfile
# Stage 1: Build your golang executable  
FROM golang as buildStage  
WORKDIR /app  
COPY . .  
RUN go build -o /go/bin/myapp  
  
# Stage 2: Run the application  
FROM alpine:latest  
COPY --from=buildStage /go/bin/myapp /usr/local/bin/myapp  
CMD ["myapp"]
```
- [Multi-stage builds 공식문서](https://docs.docker.com/build/building/multi-stage/)

### Multi-layers and caching
- `RUN`, `COPY`, `ADD` 명령어는 레이어(layer)를 만듬 
	- 다른 명령어들은 중간 임시 이미지를 생성하고 빌드 크기를 늘리지 않음. 
- 레이어는 캐시되어서, 다음에 이미지를 다시 빌드할때 변경된 레이어만 다시 빌드 => 시간 절약 
	- 파일이 추가, 수정, 삭제되어 layer의 내용물이 변경되면 캐시도 무효화 됨 
- 캐시를 타지 않게 하기 위해서는 `-nocache` 옵션 추가 
- 다중 레이어보다는 하나의 레이어가 좋다.  (`RUN` 여러개 X, 하나의 `RUN` O)
	- 예 ) `RUN yum -disablerepo=*, -enablerepo="myrepo" && yum update -y && yum install nmap` 
		- 가독성을 위해 뒤에 `&&\` 으로 다음 줄로 넘어갈 수도 있음 

### `ONBUILD` 명령어
- 부모 도커파일이 자식 도커파일에게 제공되는 명령어 
	- 하위 도커 파일의 모든 명령보다 먼저 `ONBUILD`가 먼저 실행됨 
- 부모 도커 파일을 기본 이미지로 사용하는 자식 도커파일에서 반드시 진행되어야 하는 파일 복사나, 환경 변수 설정과 같은 작업을 자동화 할 수 있다. 
- `ONBUILD` 에서 `ADD` 나 `COPY` 넣을때는 리소스가 누락되면 이미지 빌드에 실패할 수 있으니 주의해야한다. 
- 예시 ) [Ruby dockerfile](https://github.com/docker-library/ruby/blob/c43fef8a60cea31eb9e7d960a076d633cb62ba8d/2.4/jessie/onbuild/Dockerfile)

```dockerfile
FROM ruby:2.4

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

ONBUILD COPY Gemfile /usr/src/app/
ONBUILD COPY Gemfile.lock /usr/src/app/
ONBUILD RUN bundle install

ONBUILD COPY . /usr/src/app
```
- [`ONBUILD` 공식문서](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#onbuild)

### `USER`로 유저 변경하기 
- 기본적으로 컨테이너는 root로 실행. 변경하고 싶으면 `USER` 명령어 사용 가능 
- 컨테이너 실행에 있어 root를 노출시키지 않기 때문에, 보안 취약점 영향을 줄여 보안을 강화시킬 수 있다. 
```dockerfile
FROM alpine:latest  
  
# Create a new user and switch to that user  
RUN adduser -D -u 1000 appuser  
USER appuser  
  
# Set the working directory to /app  
WORKDIR /app  
  
# Copy the script to the container and make it executable  
COPY script.sh .  
RUN chmod +x script.sh  
  
# Run the script  
CMD ["./script.sh"]
```


### 컨테이너 health 체크하기 
- 컨테이너 상태 모니터링 하는 명령어 `HEALTHCHECK`
- 컨테이너가 실패했는지 확인하고 실패시 자동으로 다시 시작하도록 설정할 수 있음 

```dockerfile
# Set the command to start the web server  
CMD ["python", "app.py"]  
# Add a healthcheck for the web server  
HEALTHCHECK --interval=5s \\  
            --timeout=5s \\  
            CMD curl --fail <http://localhost:5000/health> || exit 1
```
- 예를 들어 위 dockerfile은 5초마다 로컬 5000번의 `/health` 엔드포인트를 요청하여 떠 있는 어플리케이션이 잘 동작하는지 확인 
- [`HEALTHCHECK` 공식문서](https://docs.docker.com/engine/reference/builder/#healthcheck)

### `SHELL` 사용하기
- dockerfile에서도 shell 명령어 그대로 사용 가능 
```dockerfile
SHELL ["/bin/bash", "-c"]
```

### 메타데이터 생성 
- 이미지 사용자들에게 유용한 정보 
- `EXPOSE` : 어떤 포트가 노출되는지 (실제 네트워크 영향 없음. 메타데이터 뿐임)
- `LABEL` :  이미지의 목적, 작성자 등 다양한 용도로 사용할 수 있음. 
	- `docker inspect` 로 label 쿼리 할 수 있음. 

```dockerfile
FROM ubuntu:latest  
LABEL maintainer="Your Name <youremail@example.com>"  
LABEL description="This is a simple Dockerfile example that uses the LABEL and EXPOSE instructions."  
RUN apt-get update && \  
apt-get install -y nginx  
EXPOSE 80  
CMD ["nginx", "-g", "daemon off;"]
```

### Dockerfile 빌드는 사실 그냥 tar 파일
- Docker 이미지는 사실 `tar` 파일임. 즉, 파일과 디렉토리의 압축된 아카이브 파일일 뿐인거임. 
- 이미지 빌드는 사용자가 지정한 모든 파일과 디렉토리를 가져와서 하나의 tar로 압축한 후 메타데이터를 추가 하는 것 뿐임. 

## References 
- [10 Secrets to Improve Your Dockerfile](https://aws.plainenglish.io/10-secrets-to-improve-your-dockerfile-40ac54aa5bf2)
- [Difference Between run, cmd and entrypoint in a Dockerfile](https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint)