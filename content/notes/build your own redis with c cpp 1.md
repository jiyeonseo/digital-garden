---
title: "(Build Your Own Redis with C/C++) 1. socket 통신, Protocol Parsing"
tags:
- redis
- C
---
> [Build Your Own Redis With C/C++](https://build-your-own.org/redis) 을 보고 정리한 글입니다. 더 자세한 나용은 원문을 참고하실 수 있습니다. 직접 실습한 코드는 [jiyeonseo/build-your-own-redis-with-c-cpp](https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp) 에서 찾아볼 수 있습니다.

`chap 1`은 책에 대한 introduction으로 생략. 

## Socket 동작 방식 
- Redis는 서버-클라이언트 TCP 연결을 통하여 요청/응답 수신

### 서버 workflow 수도코드
```c
fd = socket() 
bind(fd, address)
listen(fd)
while True:
    conn_fd = accept(fd)
    do_something_with(conn_fd)
    close(conn_fd)
```

- `fd` : 리눅스 커널에서 TCP 연결, 디스크 파일, 수신 포트 등을 나타내는 integer
- `bind()` : `socket()`을 통해 받은 `fd` 로 `address` 연결
- `listen()` : 해당 `address` 연결을 받을 수 있게 
- `accept()` : 수신 `fd`를 받아 연결 소켓을 반환. `conn_fd` 
- `read()` : TCP 연결에서 데이터를 수신 
- `write()` : 데이터 전송
- `close()` : `fd`가 참고하고 있는 리소스를 삭제하고 `fd` 숫자를 재활용

### 클라이언트 workflow 수도코드
```c
fd = socket()
connect(fd, address)
do_something_with(fd)
close(fd)
```
- `connect()` : 소켓 `fd` 주소를 받아 `address`와 연결

## Simple Server/Client 

### Server 
```cpp
// 먼저 소켓 `fd` 생성
int fd = socket(AF_INET, SOCK_STREAM, 0);
```
- `AF_INET` : IPv4 용. IPv6 혹은 dual-stack socket을 열 경우 `AF_INET6` 사용
- `SOCK_STREAM` : TCP 통신을 위한 stream

```cpp
// 소켓 옵션 세팅
int val = 1;
setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &val, sizeof(val));
```
- `setsockopt()` : 소켓 옵션 세팅 
	- `SO_REUSEADDR` : 서버가 재시작될때 동일한 주소 바인딩 가능
		- 소켓이 bind() 함수를 호출하여 로컬 IP 주소와 포트 번호를 할당 받을 때, 이미 그 포트를 사용 중인 다른 소켓이 있더라도 해당 포트를 재사용할 수 있도록 한다. 
		- 다시 시작시 동일 포트 번호로 사용함으로써 불필요한 대기 시간 없이 바로 다시 사용 가능.
		- 주의) 같은 포트 번호를 사용하는 두 개의 소켓이 동시에 바인드되어 있다면, 동일한 IP 주소와 포트 번호로 들어오는 패킷이 어느 소켓으로 전달될지 알 수 없게 된다.

```cpp
// bind와 listen으로 0.0.0.0:1234 바인딩 하기 
    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(0);    // wildcard address 0.0.0.0
    int rv = bind(fd, (const sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("bind()");
    }

    // listen
    rv = listen(fd, SOMAXCONN);
    if (rv) {
        die("listen()");
    }
```

```cpp
// 각각의 커넥션에 대해 do_something 하기
while (true) {
        // accept
        struct sockaddr_in client_addr = {};
        socklen_t socklen = sizeof(client_addr);
        int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
        if (connfd < 0) {
            continue;   // error
        }

        do_something(connfd);
        close(connfd);
    }
```

위 `do_something` 은 아래와 같이 read/write를 한다. 
```cpp
static void do_something(int connfd) {
    char rbuf[64] = {};
    ssize_t n = read(connfd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        msg("read() error");
        return;
    }
    printf("client says: %s\n", rbuf); // 클라이언트로부터 온 메세지 출력

    char wbuf[] = "world";
    write(connfd, wbuf, strlen(wbuf)); // 응답으로 world를 보낸다 
}
```
- `read`와 `write`가 byte 수를 반환. 

### Client 
```cpp
int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(INADDR_LOOPBACK);  // 127.0.0.1
    int rv = connect(fd, (const struct sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("connect");
    }

    char msg[] = "hello";
    write(fd, msg, strlen(msg)); // 연결되면 hello 라는 메세지를 보낸다. 

    char rbuf[64] = {};
    ssize_t n = read(fd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        die("read");
    }
    printf("server says: %s\n", rbuf);
    close(fd);
```

> 코드 : https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp/tree/main/03


## Protocol Parsing
클라이언트와 주고 받을 프로토콜을 정의해보자. 먼저 요청 길이 선언부터.
### Server
```cpp
    while (true) {
        // accept
        struct sockaddr_in client_addr = {};
        socklen_t socklen = sizeof(client_addr);
        int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
        if (connfd < 0) {
            continue;   // error
        }

        // only serves one client connection at once
        while (true) {
            int32_t err = one_request(connfd);
            if (err) {
                break;
            }
        }
        close(connfd);
    }
```
- client와 연결 후, while 룹이 돌며 하나의 요청에 대해 파씽& 응답. 

```cpp
// 실제 request처리할 `one_request` 함수에서 사용할 헬퍼 함수 2개를 먼저 만들어 본다. 
static int32_t read_full(int fd, char *buf, size_t n) {
	// n byte가 될때까지 커널에서 읽기 
	// 데이터
    while (n > 0) {
        ssize_t rv = read(fd, buf, n); 
        if (rv <= 0) {
            return -1;  // error, or unexpected EOF
        }
        assert((size_t)rv <= n);
        n -= (size_t)rv;
        buf += rv;
    }
    return 0;
}

static int32_t write_all(int fd, const char *buf, size_t n) {
    // 버퍼가 가득차면 부분적으로만 데이터를 성공적으로 쓸 수 있기 때문에 
    // 우리가 필요한 것보다 적은 bytes를 반환하도록 해준다. 
    while (n > 0) {
        ssize_t rv = write(fd, buf, n);
        if (rv <= 0) {
            return -1;  // error
        }
        assert((size_t)rv <= n);
        n -= (size_t)rv;
        buf += rv;
    }
    return 0;
}
```
위 `write`와 `read` 헬퍼 함수를 이용하여 다음과 같이 요청 처리 함수 `one_request` 를 작성할 수 있다. 

```cpp
const size_t k_max_msg = 4096;

static int32_t one_request(int connfd) {
    // 4 bytes header
    char rbuf[4 + k_max_msg + 1];
    errno = 0;
    int32_t err = read_full(connfd, rbuf, 4);
    if (err) {
        if (errno == 0) {
            msg("EOF");
        } else {
            msg("read() error");
        }
        return err;
    }

    uint32_t len = 0;
    memcpy(&len, rbuf, 4);  // assume little endian
    if (len > k_max_msg) {
        msg("too long");
        return -1;
    }

    // request body
    err = read_full(connfd, &rbuf[4], len);
    if (err) {
        msg("read() error");
        return err;
    }

    // do something
    rbuf[4 + len] = '\0';
    printf("client says: %s\n", &rbuf[4]);

    // reply using the same protocol
    const char reply[] = "world";
    char wbuf[4 + sizeof(reply)];
    len = (uint32_t)strlen(reply);
    memcpy(wbuf, &len, 4);
    memcpy(&wbuf[4], reply, len);
    return write_all(connfd, wbuf, 4 + len);
}
```
- line 7과 line 25에서 `read` 가 두 번 되고 있다. "buffered IO”를 이용하면 syscall을 줄일 수 있다. 한 번의 버퍼에 최대한 많은 양을 읽고 여러 요청을 한번에 파싱하는 방법으로 더 효율적이다. 

### Client
```cpp
static int32_t query(int fd, const char *text) {
    uint32_t len = (uint32_t)strlen(text);
    if (len > k_max_msg) {
        return -1;
    }

    char wbuf[4 + k_max_msg];
    memcpy(wbuf, &len, 4);  // assume little endian
    memcpy(&wbuf[4], text, len);
    if (int32_t err = write_all(fd, wbuf, 4 + len)) {
        return err;
    }

    // 4 bytes header
    char rbuf[4 + k_max_msg + 1];
    errno = 0;
    int32_t err = read_full(fd, rbuf, 4);
    if (err) {
        if (errno == 0) {
            msg("EOF");
        } else {
            msg("read() error");
        }
        return err;
    }

    memcpy(&len, rbuf, 4);  // assume little endian
    if (len > k_max_msg) {
        msg("too long");
        return -1;
    }

    // reply body
    err = read_full(fd, &rbuf[4], len);
    if (err) {
        msg("read() error");
        return err;
    }

    // do something
    rbuf[4 + len] = '\0';
    printf("server says: %s\n", &rbuf[4]);
    return 0;
}
```

이 장에서는 아주 간단한 len 에 대한 프로토콜만 실습하여보았지만 실제 프로토콜은 훨씬 더 복잡하다. 또 바이너리 대신 텍스트를 사용하는데, 이는 사람이 읽기 쉬운 장점이 있지만, 파씽 부분에서 바이너리보다 더 신경써야 하는 부분이 많다. 또 프로토콜에 따라 구분 부호가 달라지거나 없을 수 있기 때문에 프로토콜 분석은 실습보다 더 어려울 수 있다. 

> 코드 : https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp/tree/main/04

[[notes/build your own redis with c cpp 2]] 에 이어집니다. 

---
## Source Code 
- [jiyeonseo/build-your-own-redis-with-c-cpp](https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp)
## References 
- [Build Your Own Redis with C/C++](https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp)