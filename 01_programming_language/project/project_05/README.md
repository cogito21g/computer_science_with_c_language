# Project 05

### 1.5 간단한 웹 서버

#### 프로젝트 구조

```
simple_web_server/
├── include/
│   └── server.h
├── src/
│   ├── server.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 웹 서버

이 프로젝트는 HTTP 요청을 처리하고 간단한 HTML 응답을 제공하는 웹 서버입니다.

## 파일 구조

- `include/server.h` : 서버 함수와 구조체 선언이 포함된 헤더 파일.
- `src/server.c` : 서버 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_web_server
```

## 테스트 방법

웹 브라우저를 열고, `http://localhost:8080`에 접속하여 웹 서버의 응답을 확인합니다.
```

#### 코드 구현 및 상세 설명

**include/server.h**

```c
#ifndef SERVER_H
#define SERVER_H

#define PORT 8080
#define BUFFER_SIZE 1024

// 서버 초기화 함수
int initialize_server();

// 클라이언트 요청 처리 함수
void handle_client(int client_socket);

#endif // SERVER_H
```

**src/server.c**

```c
#include "server.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

// 서버 초기화 함수
int initialize_server() {
    int server_socket;
    struct sockaddr_in server_addr;

    // 소켓 생성
    server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket == -1) {
        perror("Socket creation failed");
        return -1;
    }

    // 서버 주소 설정
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // 소켓 바인딩
    if (bind(server_socket, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("Bind failed");
        close(server_socket);
        return -1;
    }

    // 연결 대기
    if (listen(server_socket, 10) == -1) {
        perror("Listen failed");
        close(server_socket);
        return -1;
    }

    return server_socket;
}

// 클라이언트 요청 처리 함수
void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE];
    int bytes_read;

    // 요청 읽기
    bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Read failed");
        close(client_socket);
        return;
    }

    buffer[bytes_read] = '\0';
    printf("Received request:\n%s\n", buffer);

    // HTTP 응답 작성
    const char *response =
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "Content-Length: 70\r\n"
        "\r\n"
        "<html>"
        "<head><title>Simple Web Server</title></head>"
        "<body><h1>Hello, World!</h1></body>"
        "</html>";

    // 응답 보내기
    write(client_socket, response, strlen(response));

    // 소켓 닫기
    close(client_socket);
}
```

**src/main.c**

```c
#include "server.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>

int main() {
    int server_socket, client_socket;
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);

    // 서버 초기화
    server_socket = initialize_server();
    if (server_socket == -1) {
        fprintf(stderr, "Failed to initialize server\n");
        return 1;
    }

    printf("Server is running on port %d\n", PORT);

    // 클라이언트 연결 처리
    while (1) {
        // 클라이언트 연결 수락
        client_socket = accept(server_socket, (struct sockaddr*)&client_addr, &client_addr_len);
        if (client_socket == -1) {
            perror("Accept failed");
            close(server_socket);
            return 1;
        }

        // 클라이언트 요청 처리
        handle_client(client_socket);
    }

    // 서버 소켓 닫기
    close(server_socket);
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/server.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_web_server

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`server.h`)**:
    - 서버에 필요한 함수와 상수를 정의합니다.
    - `PORT`는 웹 서버가 청취할 포트를 정의합니다.
    - `BUFFER_SIZE`는 요청을 읽고 응답을 작성할 때 사용할 버퍼의 크기를 정의합니다.
    - `initialize_server` 함수는 서버 소켓을 초기화하고 설정합니다.
    - `handle_client` 함수는 클라이언트 요청을 처리합니다.

2. **소스 파일(`server.c`)**:
    - `initialize_server` 함수는 서버 소켓을 생성하고, 주소를 설정하며, 연결 대기를 설정합니다.
    - `handle_client` 함수는 클라이언트 요청을 읽고, HTTP 응답을 작성하여 클라이언트에 전송합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 서버를 초기화하고 클라이언트 연결을 처리합니다.
    - `accept` 함수를 사용하여 클라이언트 연결을 수락하고, `handle_client` 함수를 호출하여 요청을 처리합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

### 1.5 간단한 웹 서버

#### 프로젝트 구조

```
simple_web_server/
├── include/
│   └── server.h
├── src/
│   ├── server.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 웹 서버

이 프로젝트는 기본적인 HTTP 요청을 처리하는 간단한 웹 서버입니다. 클라이언트가 접속하면 간단한 HTML 페이지를 반환합니다.

## 파일 구조

- `include/server.h` : 서버 함수와 구조체 선언이 포함된 헤더 파일.
- `src/server.c` : 서버 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_web_server <port>
```

## 사용 예

프로그램을 실행하고, 웹 브라우저에서 `http://localhost:<port>`에 접속합니다.

```sh
$ ./simple_web_server 8080
Server is running on port 8080
```
```

#### 코드 구현 및 상세 설명

**include/server.h**

```c
#ifndef SERVER_H
#define SERVER_H

#include <netinet/in.h>

#define BUFFER_SIZE 1024
#define RESPONSE_TEMPLATE "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n%s"

void start_server(int port);
void handle_client(int client_socket);

#endif // SERVER_H
```

**src/server.c**

```c
#include "server.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

void start_server(int port) {
    int server_socket, client_socket;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_len = sizeof(client_addr);

    // 소켓 생성
    server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 서버 주소 설정
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(port);

    // 소켓 바인딩
    if (bind(server_socket, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_socket);
        exit(EXIT_FAILURE);
    }

    // 수신 대기
    if (listen(server_socket, 10) < 0) {
        perror("Listen failed");
        close(server_socket);
        exit(EXIT_FAILURE);
    }

    printf("Server is running on port %d\n", port);

    while (1) {
        // 클라이언트 접속 수락
        client_socket = accept(server_socket, (struct sockaddr *)&client_addr, &client_len);
        if (client_socket < 0) {
            perror("Accept failed");
            continue;
        }

        // 클라이언트 요청 처리
        handle_client(client_socket);
        close(client_socket);
    }

    close(server_socket);
}

void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE];
    int bytes_read;
    char response[BUFFER_SIZE];

    // 클라이언트 요청 읽기
    bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Read failed");
        return;
    }
    buffer[bytes_read] = '\0';

    // 간단한 HTML 페이지 생성
    const char *html = "<html><body><h1>Welcome to Simple Web Server!</h1></body></html>";
    snprintf(response, sizeof(response), RESPONSE_TEMPLATE, html);

    // 클라이언트에 응답 보내기
    write(client_socket, response, strlen(response));
}
```

**src/main.c**

```c
#include "server.h"
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <port>\n", argv[0]);
        return 1;
    }

    int port = atoi(argv[1]);
    start_server(port);

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/server.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_web_server

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`server.h`)**:
    - 서버에 필요한 함수와 상수의 선언이 포함되어 있습니다.
    - `BUFFER_SIZE`는 버퍼 크기를 정의합니다.
    - `RESPONSE_TEMPLATE`는 HTTP 응답 템플릿을 정의합니다.
    - `start_server` 함수는 서버를 시작합니다.
    - `handle_client` 함수는 클라이언트 요청을 처리합니다.

2. **소스 파일(`server.c`)**:
    - `start_server` 함수는 서버를 시작하고 클라이언트 요청을 처리합니다.
        - 소켓을 생성하고, 서버 주소를 설정하고, 소켓을 바인딩합니다.
        - 수신 대기 상태로 전환하고 클라이언트의 접속을 수락합니다.
        - `handle_client` 함수를 호출하여 클라이언트 요청을 처리합니다.
    - `handle_client` 함수는 클라이언트 요청을 읽고, 간단한 HTML 페이지를 응답합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 포트 번호를 받습니다.
    - `start_server` 함수를 호출하여 웹 서버를 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
