---
title: "(Build Your Own Redis with C/C++) 2. Event Loop"
tags:
- redis
- C
---
> [Build Your Own Redis With C/C++](https://build-your-own.org/redis) 을 보고 정리한 글입니다. 더 자세한 나용은 원문을 참고하실 수 있습니다. 직접 실습한 코드는 [jiyeonseo/build-your-own-redis-with-c-cpp](https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp) 에서 찾아볼 수 있습니다. [[notes/build your own redis with c cpp 1]] 이어지는 노트입니다. 

## The Event Loop and Nonblocking IO
서버와의 네트워킹 동시 연결 처리 방법 3가지 
- Forking : 클라이언트와의 연결마다 새로운 프로세스 만드는 방법 
- Multi-threading : 프로세스 대신 여러 쓰레드를 사용하는 방법  
- Event loops : 폴링과 논블로킹 IO를 이용하는 것으로 주로 단일 쓰레드위에서 실행. 프로세스와 쓰레드 오버헤드로 인해 대부분 이벤트 룹을 많이 사용하는 추세. 

event loop을 이용한 서버쪽은 다음과 같이 표현할 수 있다. 
```
all_fds = [...]
while True:
    active_fds = poll(all_fds) # blocking 없이 바로 작동 가능한지 확인 
    for each fd in active_fds:
        do_something_with(fd)

def do_something_with(fd):
    if fd is a listening socket:
        add_new_client(fd)
    elif fd is a client connection:
        while work_not_done(fd):
            do_something_to_client(fd)

def do_something_to_client(fd):
    if should_read_from(fd):
        data = read_until_EAGAIN(fd)
        process_incoming_data(data)
    while should_write_to(fd):
        write_until_EAGAIN(fd)
    if should_close(fd):
        destroy_client(fd)
```
- `fd` 로 IO 작업을 하기 위해서는 nonblocking 모드여야 함
	- blocking 모드에서는 아래와 같은 경우 block 된다. 
		- `read`  실행시 커널에 데이터가 없으면
		- `write` 실행시 버퍼가 가즉 찼으면 
		- `accept` 실행시 커널 큐에 커넥션이 없으면 
	- nonblocking 모드에서는, 그냥 성공 하거나 아직 준비 안됨 오류인 `EAGAIN` 오류를 내며 실패. 
		- `EAGAIN` 오류가 나면 `poll` 을 통해 준비가 다 끝난 후 다시 시도해야만 함. 
- `poll` : 이벤트 룹에서의 유일한 blocking 작업. 
- 논블락킹 모드에서 blocking 밖에 지원하지 않는 API (예를 들어 `gethostbyname` 이나 disk IO) 는 쓰레드 풀에서 수행해야만 한다. 
- `fd`를 논블락킹 모드로 세팅하기 위해서는 `fcntl`을 사용하면 된다. 
```cpp
static void fd_set_nb(int fd) {
    errno = 0;
    int flags = fcntl(fd, F_GETFL, 0);
    if (errno) {
        die("fcntl error");
        return;
    }

    flags |= O_NONBLOCK;

    errno = 0;
    (void)fcntl(fd, F_SETFL, flags);
    if (errno) {
        die("fcntl error");
    }
}
```
- Linux에서는 `poll` 이외에도 `select`, `epoll`과 같은 syscall들이 있다. 
	- `select` : 기본적으로는 `poll`과 비슷. 하지만 최대 `fd` 제한수가 적어 요즘에는 사용되고 있지 않음. 
	- `epoll` : stateful API. 
		- `epoll_create` : `fd` 생성
		- `epoll_wait` : `fd` 작동 
		- `epoll_ctl` : `fd` 조작

위 Event loop을 이용해서 서버 코드를 업데이트 해보자. 먼저 `Conn`.   
```cpp
enum {
    STATE_REQ = 0,
    STATE_RES = 1,
    STATE_END = 2,  // mark the connection for deletion
};

struct Conn {
    int fd = -1;
    uint32_t state = 0;     // either STATE_REQ or STATE_RES
    // buffer for reading
    size_t rbuf_size = 0;
    uint8_t rbuf[4 + k_max_msg];
    // buffer for writing
    size_t wbuf_size = 0;
    size_t wbuf_sent = 0;
    uint8_t wbuf[4 + k_max_msg];
};
```
- nonblocking 모드에서 IO 작업(read, write)를 위한 버퍼 필요 
- `state` connection 상태용 
	- `STATE_REQ` : reqding request 용 
	- `STATE_RES` : sending response 용

```cpp 
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    // bind, listen and etc
    // code omitted...

    // a map of all client connections, keyed by fd
    std::vector<Conn *> fd2conn;

    // set the listen fd to nonblocking mode
    fd_set_nb(fd);

    // the event loop
    std::vector<struct pollfd> poll_args;
    while (true) {
        // prepare the arguments of the poll()
        poll_args.clear();
        // for convenience, the listening fd is put in the first position
        struct pollfd pfd = {fd, POLLIN, 0};
        poll_args.push_back(pfd);
        // connection fds
        for (Conn *conn : fd2conn) {
            if (!conn) {
                continue;
            }
            struct pollfd pfd = {};
            pfd.fd = conn->fd;
            pfd.events = (conn->state == STATE_REQ) ? POLLIN : POLLOUT;
            pfd.events = pfd.events | POLLERR;
            poll_args.push_back(pfd);
        }

        // poll for active fds
        // the timeout argument doesn't matter here
        int rv = poll(poll_args.data(), (nfds_t)poll_args.size(), 1000);
        if (rv < 0) {
            die("poll");
        }

        // process active connections
        for (size_t i = 1; i < poll_args.size(); ++i) {
            if (poll_args[i].revents) {
                Conn *conn = fd2conn[poll_args[i].fd];
                connection_io(conn);
                if (conn->state == STATE_END) {
                    // client closed normally, or something bad happened.
                    // destroy this connection
                    fd2conn[conn->fd] = NULL;
                    (void)close(conn->fd);
                    free(conn);
                }
            }
        }

        // try to accept a new connection if the listening fd is active
        if (poll_args[0].revents) {
            (void)accept_new_conn(fd2conn, fd);
        }
    }
```
- 이벤트 룹에서 가장 먼저 필요한 것은 `poll` 설정
	- listening `fd`  :  `POLLIN`  상태로 
	- connection `fd` :  struct `Conn` 에 따라 상태가 변경 
- 때에 따라, poll 상태가 reading(`POLLIN`) 이거나 writing (`POLLOUT`)이 아닌 경우도 있다. 
- `epoll`을 사용한다면 `epoll_ctl` 로 설정된 `fd` 로 업데이트 해주어야 한다. 
- `poll` argument로 timeout 값 설정도 있는데 여기서는 크게 중요하지 않음으로 임의의 큰 수를 넣어줌. 
- `poll`을 반환 받으면 reading/writing 준비가 될때 마다 알림을 받을 수 있다. 
- `accept_new_conn` 은 새로운 connection을 받고 `struct Conn` 객체를 만든다. (아래)

```cpp
static int32_t accept_new_conn(std::vector<Conn *> &fd2conn, int fd) {
    // accept 
    struct sockaddr_in client_addr = {};
    socklen_t socklen = sizeof(client_addr);
    int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
    if (connfd < 0) {
        msg("accept() error");
        return -1;  // error
    }

    // set the new connection fd to nonblocking mode
    fd_set_nb(connfd);
    // creating the struct Conn
    struct Conn *conn = (struct Conn *)malloc(sizeof(struct Conn));
    if (!conn) {
        close(connfd);
        return -1;
    }
    conn->fd = connfd;
    conn->state = STATE_REQ;
    conn->rbuf_size = 0;
    conn->wbuf_size = 0;
    conn->wbuf_sent = 0;
    conn_put(fd2conn, conn);
    return 0;
}
```
- 클라이언트 연결 상태 체크(state machine)를 위한 `connection_io()`
```cpp
static void connection_io(Conn *conn) {
    if (conn->state == STATE_REQ) {
        state_req(conn);
    } else if (conn->state == STATE_RES) {
        state_res(conn);
    } else {
        assert(0);  // not expected
    }
}
```
- `STATE_REQ` 는 읽기 상태 
```cpp
static void state_req(Conn *conn) {
    while (try_fill_buffer(conn)) {}
}

static bool try_fill_buffer(Conn *conn) {
    // try to fill the buffer
    assert(conn->rbuf_size < sizeof(conn->rbuf));
    ssize_t rv = 0;
    do {
        size_t cap = sizeof(conn->rbuf) - conn->rbuf_size;
        rv = read(conn->fd, &conn->rbuf[conn->rbuf_size], cap);
    } while (rv < 0 && errno == EINTR);
    if (rv < 0 && errno == EAGAIN) {
        // got EAGAIN, stop.
        return false;
    }
    if (rv < 0) {
        msg("read() error");
        conn->state = STATE_END;
        return false;
    }
    if (rv == 0) {
        if (conn->rbuf_size > 0) {
            msg("unexpected EOF");
        } else {
            msg("EOF");
        }
        conn->state = STATE_END;
        return false;
    }

    conn->rbuf_size += (size_t)rv;
    assert(conn->rbuf_size <= sizeof(conn->rbuf) - conn->rbuf_size);

    // Try to process requests one by one.
    // Why is there a loop? Please read the explanation of "pipelining".
    while (try_one_request(conn)) {}
    return (conn->state == STATE_REQ);
}
```
- `try_fill_buffer` : read buffer를 데이터로 채워주는 함수 
	- read buffer가 제한적이기 때문에 만약 꽉 차게 되면 그 전에 `EAGAIN` 에러를 발생시킨다. 
	- 따라서, 읽기 후 read buffer를 처리하고 `EAGAIN` 발생 전까지 룹을 돈다. 
- `read` syscall은 `EINTR` 에러 후에 리트라이 해주어야 한다.
	- `EINTR`은 syscall이 중단 되었다는 뜻으로, 어플리케이션과 상관 없이 재시도를 해주어야 한다. 
- `try_fill_buffer` 에서 loop을 도는 이유 
	- 요청/응답 프로토콜에서 클라이언트가 요청 보내놓고 계속 기다리는 것이 아니라 계속 필요한 요청을 계속 보낼 수 있으면 시간을 더 절약할 수 있음. -> "pipelining" 
	- 따라서, read buffer에 하나의 요청만 포함된다 라고 할 수 없음. 하나의 read buffer에는 여러 요청이 있을 수 있음. 
- `try_one_request` 에서 read buffer에 있는 요청 하나씩을 꺼내와 응답을 만들고 `STATE_RES` 상태값으로 변경한다. 
```cpp
static bool try_one_request(Conn *conn) {
    // try to parse a request from the buffer
    if (conn->rbuf_size < 4) {
        // not enough data in the buffer. Will retry in the next iteration
        return false;
    }
    uint32_t len = 0;
    memcpy(&len, &conn->rbuf[0], 4);
    if (len > k_max_msg) {
        msg("too long");
        conn->state = STATE_END;
        return false;
    }
    if (4 + len > conn->rbuf_size) {
        // not enough data in the buffer. Will retry in the next iteration
        return false;
    }

    // got one request, do something with it
    printf("client says: %.*s\n", len, &conn->rbuf[4]);

    // generating echoing response
    memcpy(&conn->wbuf[0], &len, 4);
    memcpy(&conn->wbuf[4], &conn->rbuf[4], len);
    conn->wbuf_size = 4 + len;

    // remove the request from the buffer.
    // note: frequent memmove is inefficient.
    // note: need better handling for production code.
    size_t remain = conn->rbuf_size - 4 - len;
    if (remain) {
        memmove(conn->rbuf, &conn->rbuf[4 + len], remain);
    }
    conn->rbuf_size = remain;

    // change state
    conn->state = STATE_RES;
    state_res(conn);

    // continue the outer loop if the request was fully processed
    return (conn->state == STATE_REQ);
}
```

변경된 conn을 받는 `state_res` 는 아래와 같다.
- `EAGAIN` 이 될때까지 계속 write buffer를 flush 하고, flush가 완료되면 state를 `STATE_REQ`로 바꿔준다.  

```cpp
static void state_res(Conn *conn) {
    while (try_flush_buffer(conn)) {}
}

static bool try_flush_buffer(Conn *conn) {
    ssize_t rv = 0;
    do {
        size_t remain = conn->wbuf_size - conn->wbuf_sent;
        rv = write(conn->fd, &conn->wbuf[conn->wbuf_sent], remain);
    } while (rv < 0 && errno == EINTR);
    if (rv < 0 && errno == EAGAIN) {
        // EAGAIN 이 되면 멈춘다.
        return false;
    }
    if (rv < 0) {
        msg("write() error");
        conn->state = STATE_END;
        return false;
    }
    conn->wbuf_sent += (size_t)rv;
    assert(conn->wbuf_sent <= conn->wbuf_size);
    if (conn->wbuf_sent == conn->wbuf_size) {
        // response가 모두 보내지면, STATE_REQ로 다시 state를 바꾸어 준다.
        conn->state = STATE_REQ;
        conn->wbuf_sent = 0;
        conn->wbuf_size = 0;
        return false;
    }
    // 아직 write buffer(wbuf)에 데이터가 남아있다면 다시 쓰기를 시도한다.
    return true;
}
```

> 코드 : https://github.com/jiyeonseo/build-your-own-redis-with-c-cpp/tree/main/06