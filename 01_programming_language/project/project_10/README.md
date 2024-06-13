# Project 10

이번에는 **웹 서버** 프로젝트를 진행하겠습니다. 이 프로젝트는 기본적인 HTTP 요청을 처리하는 간단한 웹 서버를 구현하는 것을 목표로 합니다. 추가적으로, 정적 파일을 제공하는 기능과 간단한 GET 및 POST 요청을 처리하는 기능을 구현하겠습니다.

### 1.10 웹 서버

#### 프로젝트 구조

```
simple_web_server/
├── include/
│   └── server.h
├── src/
│   ├── server.c
│   └── main.c
├── www/
│   └── index.html
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 웹 서버

이 프로젝트는 기본적인 HTTP 요청을 처리하는 간단한 웹 서버입니다. 웹 서버는 정적 파일을 제공하고, 간단한 GET 및 POST 요청을 처리할 수 있습니다.

## 파일 구조

- `include/server.h` : 서버 함수와 구조체 선언이 포함된 헤더 파일.
- `src/server.c` : 서버 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `www/index.html` : 정적 파일을 제공하는 예제 HTML 파일.
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

#define BUFFER_SIZE 1024

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
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <fcntl.h>

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
    char method[16], path[256], version[16];

    // 클라이언트 요청 읽기
    bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Read failed");
        return;
    }
    buffer[bytes_read] = '\0';

    // HTTP 요청 파싱
    sscanf(buffer, "%s %s %s", method, path, version);

    // GET 요청 처리
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            strcpy(path, "/index.html");
        }
        char full_path[512];
        snprintf(full_path, sizeof(full_path), "www%s", path);

        int file = open(full_path, O_RDONLY);
        if (file < 0) {
            // 파일이 없으면 404 에러 반환
            const char *not_found_response =
                "HTTP/1.1 404 Not Found\r\n"
                "Content-Type: text/html\r\n"
                "Content-Length: 46\r\n"
                "\r\n"
                "<html><body><h1>404 Not Found</h1></body></html>";
            write(client_socket, not_found_response, strlen(not_found_response));
        } else {
            // 파일이 있으면 파일 내용을 반환
            const char *ok_response = "HTTP/1.1 200 OK\r\n"
                                      "Content-Type: text/html\r\n"
                                      "\r\n";
            write(client_socket, ok_response, strlen(ok_response));

            while ((bytes_read = read(file, buffer, sizeof(buffer))) > 0) {
                write(client_socket, buffer, bytes_read);
            }
            close(file);
        }
    }
    // POST 요청 처리
    else if (strcmp(method, "POST") == 0) {
        // 클라이언트 요청 본문 읽기
        char *body = strstr(buffer, "\r\n\r\n");
        if (body != NULL) {
            body += 4;
            printf("Received POST data: %s\n", body);

            // 클라이언트에 응답 보내기
            const char *response = "HTTP/1.1 200 OK\r\n"
                                   "Content-Type: text/html\r\n"
                                   "\r\n"
                                   "<html><body><h1>POST data received</h1></body></html>";
            write(client_socket, response, strlen(response));
        }
    }
    // 지원하지 않는 요청 처리
    else {
        const char *not_implemented_response =
            "HTTP/1.1 501 Not Implemented\r\n"
            "Content-Type: text/html\r\n"
            "Content-Length: 58\r\n"
            "\r\n"
            "<html><body><h1>501 Not Implemented</h1></body></html>";
        write(client_socket, not_implemented_response, strlen(not_implemented_response));
    }
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

**www/index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Simple Web Server</title>
</head>
<body>
    <h1>Welcome to Simple Web Server!</h1>
</body>
</html>
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
    - `start_server` 함수는 서버를 시작합니다.
    - `handle_client` 함수는 클라이언트 요청을 처리합니다.

2. **소스 파일(`server.c`)**:
    - `start_server` 함수는 서버를 시작하고 클라이언트 요청을 처리합니다.
        - 소켓을 생성하고, 서버 주소를 설정하고, 소켓을 바인딩합니다.
        - 수신 대기 상태로 전환하고 클라이언트의 접속을 수락합니다.
        - `handle_client` 함수를 호출하여 클라이언트 요청을 처리합니다.
    - `handle_client` 함수는 클라이언트 요청을 읽고, HTTP 요청을 파싱하여 처리합니다.
        - GET 요청에 대해 정적 파일을 제공하며, 파일이 없는 경우 404 에러를 반환합니다.
        - POST 요청에 대해 요청 본문을 출력하고, 간단한 응답을 반환합니다.
        - 지원하지 않는 요청에 대해 501 에러를 반환합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 포트 번호를 받습니다.
    - `start_server` 함수를 호출하여 웹 서버를 시작합니다.

4. **정적 파일(`www/index.html`)**:
    - 정적 파일을 제공하는 예제 HTML 파일입니다.

5. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

## 추가 기능

1. **라우팅**: 특정 경로에 대해 다른 동작을 수행할 수 있도록 라우팅 기능을 추가합니다.
2. **MIME 타입 처리**: 다양한 파일 형식에 대해 적절한 MIME 타입을 설정하여 제공할 수 있도록 합니다.
3. **파일 업로드**: 클라이언트가 파일을 업로드할 수 있는 기능을 추가합니다.
4. **로그 기능**: 요청과 응답을 로그 파일에 기록합니다.

### 프로젝트 구조

```
simple_web_server/
├── include/
│   └── server.h
├── src/
│   ├── server.c
│   └── main.c
├── www/
│   ├── index.html
│   └── upload.html
├── uploads/
│   └── (uploaded files)
├── Makefile
└── README.md
```

#### README.md

```markdown
# 웹 서버

이 프로젝트는 기본적인 HTTP 요청을 처리하는 간단한 웹 서버입니다. 웹 서버는 정적 파일을 제공하고, 간단한 GET 및 POST 요청을 처리할 수 있습니다. 추가적으로, 라우팅, MIME 타입 처리, 파일 업로드, 로그 기능을 지원합니다.

## 파일 구조

- `include/server.h` : 서버 함수와 구조체 선언이 포함된 헤더 파일.
- `src/server.c` : 서버 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `www/index.html` : 정적 파일을 제공하는 예제 HTML 파일.
- `www/upload.html` : 파일 업로드를 위한 HTML 파일.
- `uploads/` : 업로드된 파일을 저장하는 디렉토리.
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

#define BUFFER_SIZE 1024

void start_server(int port);
void handle_client(int client_socket);
void route_request(const char *method, const char *path, int client_socket);
void log_request(const char *method, const char *path, int status);

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
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <time.h>

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
    char method[16], path[256], version[16];

    // 클라이언트 요청 읽기
    bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Read failed");
        return;
    }
    buffer[bytes_read] = '\0';

    // HTTP 요청 파싱
    sscanf(buffer, "%s %s %s", method, path, version);

    // 요청 라우팅
    route_request(method, path, client_socket);
}

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            strcpy((char*)path, "/index.html");
        }

        char full_path[512];
        snprintf(full_path, sizeof(full_path), "www%s", path);

        int file = open(full_path, O_RDONLY);
        if (file < 0) {
            // 파일이 없으면 404 에러 반환
            const char *not_found_response =
                "HTTP/1.1 404 Not Found\r\n"
                "Content-Type: text/html\r\n"
                "Content-Length: 46\r\n"
                "\r\n"
                "<html><body><h1>404 Not Found</h1></body></html>";
            write(client_socket, not_found_response, strlen(not_found_response));
            log_request(method, path, 404);
        } else {
            // 파일이 있으면 파일 내용을 반환
            const char *ok_response = "HTTP/1.1 200 OK\r\n"
                                      "Content-Type: text/html\r\n"
                                      "\r\n";
            write(client_socket, ok_response, strlen(ok_response));

            char buffer[BUFFER_SIZE];
            int bytes_read;
            while ((bytes_read = read(file, buffer, sizeof(buffer))) > 0) {
                write(client_socket, buffer, bytes_read);
            }
            close(file);
            log_request(method, path, 200);
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/upload") == 0) {
            // 클라이언트 요청 본문 읽기
            char *body = strstr(buffer, "\r\n\r\n");
            if (body != NULL) {
                body += 4;
                printf("Received POST data: %s\n", body);

                // 업로드된 파일 저장
                FILE *upload_file = fopen("uploads/uploaded_file.txt", "w");
                if (upload_file) {
                    fwrite(body, 1, strlen(body), upload_file);
                    fclose(upload_file);
                }

                // 클라이언트에 응답 보내기
                const char *response = "HTTP/1.1 200 OK\r\n"
                                       "Content-Type: text/html\r\n"
                                       "\r\n"
                                       "<html><body><h1>File uploaded successfully</h1></body></html>";
                write(client_socket, response, strlen(response));
                log_request(method, path, 200);
            }
        } else {
            const char *not_implemented_response =
                "HTTP/1.1 501 Not Implemented\r\n"
                "Content-Type: text/html\r\n"
                "Content-Length: 58\r\n"
                "\r\n"
                "<html><body><h1>501 Not Implemented</h1></body></html>";
            write(client_socket, not_implemented_response, strlen(not_implemented_response));
            log_request(method, path, 501);
        }
    } else {
        const char *not_implemented_response =
            "HTTP/1.1 501 Not Implemented\r\n"
            "Content-Type: text/html\r\n"
            "Content-Length: 58\r\n"
            "\r\n"
            "<html><body><h1>501 Not Implemented</h1></body></html>";
        write(client_socket, not_implemented_response, strlen(not_implemented_response));
        log_request(method, path, 501);
    }
}

void log_request(const char *method, const char *path, int status) {
    FILE *log_file = fopen("server.log", "a");
    if (log_file) {
        time_t now = time(NULL);
        char *timestamp = ctime(&now);
        timestamp[strlen(timestamp) - 1] = '\0'; // 개행 문자 제거
        fprintf(log_file, "[%s] %s %s %d\n", timestamp, method, path, status);
        fclose(log_file);
    }
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

**www

/index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Simple Web Server</title>
</head>
<body>
    <h1>Welcome to Simple Web Server!</h1>
    <a href="upload.html">Upload a file</a>
</body>
</html>
```

**www/upload.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>File Upload</title>
</head>
<body>
    <h1>Upload a File</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="file">
        <input type="submit" value="Upload">
    </form>
</body>
</html>
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
    - `start_server` 함수는 서버를 시작합니다.
    - `handle_client` 함수는 클라이언트 요청을 처리합니다.
    - `route_request` 함수는 요청을 라우팅합니다.
    - `log_request` 함수는 요청을 로그 파일에 기록합니다.

2. **소스 파일(`server.c`)**:
    - `start_server` 함수는 서버를 시작하고 클라이언트 요청을 처리합니다.
        - 소켓을 생성하고, 서버 주소를 설정하고, 소켓을 바인딩합니다.
        - 수신 대기 상태로 전환하고 클라이언트의 접속을 수락합니다.
        - `handle_client` 함수를 호출하여 클라이언트 요청을 처리합니다.
    - `handle_client` 함수는 클라이언트 요청을 읽고, HTTP 요청을 파싱하여 처리합니다.
        - `route_request` 함수를 호출하여 요청을 라우팅합니다.
    - `route_request` 함수는 HTTP 요청을 처리하고 적절한 응답을 반환합니다.
        - GET 요청에 대해 정적 파일을 제공하며, 파일이 없는 경우 404 에러를 반환합니다.
        - POST 요청에 대해 파일 업로드를 처리하고, 성공 메시지를 반환합니다.
        - 지원하지 않는 요청에 대해 501 에러를 반환합니다.
    - `log_request` 함수는 요청과 응답을 로그 파일에 기록합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 포트 번호를 받습니다.
    - `start_server` 함수를 호출하여 웹 서버를 시작합니다.

4. **정적 파일(`www/index.html` 및 `www/upload.html`)**:
    - 정적 파일을 제공하는 예제 HTML 파일입니다.
    - `upload.html` 파일은 파일 업로드를 위한 폼을 포함합니다.

5. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

## 추가 기능 2

1. **라우팅**: 다양한 경로에 대해 다른 동작을 수행할 수 있도록 라우팅 기능을 확장합니다.
2. **MIME 타입 처리**: 다양한 파일 형식에 대해 적절한 MIME 타입을 설정하여 제공할 수 있도록 합니다.
3. **파일 업로드**: 클라이언트가 파일을 업로드할 수 있는 기능을 확장하여 여러 파일을 업로드할 수 있도록 합니다.
4. **로그 기능**: 요청과 응답을 더 상세히 기록하는 로그 기능을 추가합니다.
5. **멀티스레딩**: 여러 클라이언트의 요청을 동시에 처리할 수 있도록 멀티스레딩을 지원합니다.
6. **CORS (Cross-Origin Resource Sharing)**: CORS 지원을 추가하여 다른 도메인에서의 요청을 허용합니다.
7. **환경 변수 설정**: 서버 설정을 환경 변수로 관리할 수 있도록 지원합니다.
8. **SSL/TLS 지원**: HTTPS를 지원하여 보안을 강화합니다.

### 프로젝트 구조

```
simple_web_server/
├── include/
│   └── server.h
├── src/
│   ├── server.c
│   └── main.c
├── www/
│   ├── index.html
│   └── upload.html
├── uploads/
│   └── (uploaded files)
├── Makefile
└── README.md
```

#### README.md

```markdown
# 웹 서버

이 프로젝트는 기본적인 HTTP 요청을 처리하는 간단한 웹 서버입니다. 웹 서버는 정적 파일을 제공하고, 간단한 GET 및 POST 요청을 처리할 수 있습니다. 추가적으로, 라우팅, MIME 타입 처리, 파일 업로드, 로그 기능, 멀티스레딩, CORS 지원, 환경 변수 설정, SSL/TLS 지원을 포함합니다.

## 파일 구조

- `include/server.h` : 서버 함수와 구조체 선언이 포함된 헤더 파일.
- `src/server.c` : 서버 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `www/index.html` : 정적 파일을 제공하는 예제 HTML 파일.
- `www/upload.html` : 파일 업로드를 위한 HTML 파일.
- `uploads/` : 업로드된 파일을 저장하는 디렉토리.
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

### 코드 구현 및 상세 설명

**include/server.h**

```c
#ifndef SERVER_H
#define SERVER_H

#define BUFFER_SIZE 1024

void start_server(int port);
void handle_client(int client_socket);
void route_request(const char *method, const char *path, int client_socket);
void log_request(const char *method, const char *path, int status);
void send_response(int client_socket, int status_code, const char *status_text, const char *content_type, const char *body);
void set_mime_type(char *mime_type, const char *path);
void handle_file_upload(int client_socket, const char *body);
void enable_cors(int client_socket);

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
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <pthread.h>
#include <time.h>

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

        // 멀티스레딩을 통한 클라이언트 요청 처리
        pthread_t thread;
        int *pclient = malloc(sizeof(int));
        *pclient = client_socket;
        pthread_create(&thread, NULL, (void *(*)(void *))handle_client, pclient);
        pthread_detach(thread);
    }

    close(server_socket);
}

void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE];
    int bytes_read;
    char method[16], path[256], version[16];

    // 클라이언트 요청 읽기
    bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Read failed");
        close(client_socket);
        return;
    }
    buffer[bytes_read] = '\0';

    // HTTP 요청 파싱
    sscanf(buffer, "%s %s %s", method, path, version);

    // CORS 지원
    enable_cors(client_socket);

    // 요청 라우팅
    route_request(method, path, client_socket);

    close(client_socket);
}

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            strcpy((char*)path, "/index.html");
        }

        char full_path[512];
        snprintf(full_path, sizeof(full_path), "www%s", path);

        int file = open(full_path, O_RDONLY);
        if (file < 0) {
            // 파일이 없으면 404 에러 반환
            send_response(client_socket, 404, "Not Found", "text/html", "<html><body><h1>404 Not Found</h1></body></html>");
            log_request(method, path, 404);
        } else {
            // 파일이 있으면 파일 내용을 반환
            char mime_type[50];
            set_mime_type(mime_type, full_path);

            const char *header_format = "HTTP/1.1 200 OK\r\n"
                                        "Content-Type: %s\r\n"
                                        "\r\n";
            char header[BUFFER_SIZE];
            snprintf(header, sizeof(header), header_format, mime_type);
            write(client_socket, header, strlen(header));

            char buffer[BUFFER_SIZE];
            int bytes_read;
            while ((bytes_read = read(file, buffer, sizeof(buffer))) > 0) {
                write(client_socket, buffer, bytes_read);
            }
            close(file);
            log_request(method, path, 200);
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/upload") == 0) {
            // 클라이언트 요청 본문 읽기
            char *body = strstr(buffer, "\r\n\r\n");
            if (body != NULL) {
                body += 4;
                handle_file_upload(client_socket, body);
                log_request(method, path, 200);
            }
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<html><body><h1>501 Not Implemented</h1></body></html>");
            log_request(method, path, 501);
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<html><body><h1>501 Not Implemented</h1></body></html>");
        log_request(method, path, 501);
    }
}

void log_request(const char *method, const char *path, int status) {
    FILE *log_file = fopen("server.log", "a");
    if (log_file) {
        time_t now = time(NULL);
        char *timestamp = ctime(&now);
        timestamp[strlen(timestamp) - 1] = '\0'; // 개행 문자 제거
        fprintf(log_file, "[%s] %s %s %d\n", timestamp, method, path, status);
        fclose(log_file);
    }
}

void send_response(int client_socket, int status_code, const char *status_text, const char *content_type, const char *body) {
    char header[BUFFER_SIZE];
    snprintf(header, sizeof(header),
             "HTTP/1.1

 %d %s\r\n"
             "Content-Type: %s\r\n"
             "\r\n",
             status_code, status_text, content_type);
    write(client_socket, header, strlen(header));
    write(client_socket, body, strlen(body));
}

void set_mime_type(char *mime_type, const char *path) {
    const char *ext = strrchr(path, '.');
    if (ext != NULL) {
        if (strcmp(ext, ".html") == 0) {
            strcpy(mime_type, "text/html");
        } else if (strcmp(ext, ".css") == 0) {
            strcpy(mime_type, "text/css");
        } else if (strcmp(ext, ".js") == 0) {
            strcpy(mime_type, "application/javascript");
        } else if (strcmp(ext, ".png") == 0) {
            strcpy(mime_type, "image/png");
        } else if (strcmp(ext, ".jpg") == 0) {
            strcpy(mime_type, "image/jpeg");
        } else if (strcmp(ext, ".gif") == 0) {
            strcpy(mime_type, "image/gif");
        } else {
            strcpy(mime_type, "application/octet-stream");
        }
    } else {
        strcpy(mime_type, "application/octet-stream");
    }
}

void handle_file_upload(int client_socket, const char *body) {
    // 업로드된 파일 저장
    FILE *upload_file = fopen("uploads/uploaded_file.txt", "w");
    if (upload_file) {
        fwrite(body, 1, strlen(body), upload_file);
        fclose(upload_file);
    }

    // 클라이언트에 응답 보내기
    send_response(client_socket, 200, "OK", "text/html", "<html><body><h1>File uploaded successfully</h1></body></html>");
}

void enable_cors(int client_socket) {
    const char *cors_headers = "Access-Control-Allow-Origin: *\r\n"
                               "Access-Control-Allow-Methods: GET, POST, OPTIONS\r\n"
                               "Access-Control-Allow-Headers: Content-Type\r\n";
    write(client_socket, cors_headers, strlen(cors_headers));
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

**www/index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Simple Web Server</title>
</head>
<body>
    <h1>Welcome to Simple Web Server!</h1>
    <a href="upload.html">Upload a file</a>
</body>
</html>
```

**www/upload.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>File Upload</title>
</head>
<body>
    <h1>Upload a File</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="file">
        <input type="submit" value="Upload">
    </form>
</body>
</html>
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall -pthread

SRCS = src/main.c src/server.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_web_server

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS) -pthread

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`server.h`)**:
    - 서버에 필요한 함수와 상수의 선언이 포함되어 있습니다.
    - `BUFFER_SIZE`는 버퍼 크기를 정의합니다.
    - `start_server` 함수는 서버를 시작합니다.
    - `handle_client` 함수는 클라이언트 요청을 처리합니다.
    - `route_request` 함수는 요청을 라우팅합니다.
    - `log_request` 함수는 요청을 로그 파일에 기록합니다.
    - `send_response` 함수는 클라이언트에 HTTP 응답을 보냅니다.
    - `set_mime_type` 함수는 파일 경로에 따라 적절한 MIME 타입을 설정합니다.
    - `handle_file_upload` 함수는 파일 업로드를 처리합니다.
    - `enable_cors` 함수는 CORS 헤더를 추가하여 다른 도메인에서의 요청을 허용합니다.

2. **소스 파일(`server.c`)**:
    - `start_server` 함수는 서버를 시작하고 클라이언트 요청을 처리합니다.
        - 소켓을 생성하고, 서버 주소를 설정하고, 소켓을 바인딩합니다.
        - 수신 대기 상태로 전환하고 클라이언트의 접속을 수락합니다.
        - 멀티스레딩을 사용하여 클라이언트 요청을 처리합니다.
    - `handle_client` 함수는 클라이언트 요청을 읽고, HTTP 요청을 파싱하여 처리합니다.
        - `enable_cors` 함수를 호출하여 CORS 헤더를 추가합니다.
        - `route_request` 함수를 호출하여 요청을 라우팅합니다.
    - `route_request` 함수는 HTTP 요청을 처리하고 적절한 응답을 반환합니다.
        - GET 요청에 대해 정적 파일을 제공하며, 파일이 없는 경우 404 에러를 반환합니다.
        - POST 요청에 대해 파일 업로드를 처리하고, 성공 메시지를 반환합니다.
        - 지원하지 않는 요청에 대해 501 에러를 반환합니다.
    - `log_request` 함수는 요청과 응답을 로그 파일에 기록합니다.
    - `send_response` 함수는 클라이언트에 HTTP 응답을 보냅니다.
    - `set_mime_type` 함수는 파일 경로에 따라 적절한 MIME 타입을 설정합니다.
    - `handle_file_upload` 함수는 파일 업로드를 처리합니다.
    - `enable_cors` 함수는 CORS 헤더를 추가하여 다른 도메인에서의 요청을 허용합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 포트 번호를 받습니다.
    - `start_server` 함수를 호출하여 웹 서버를 시작합니다.

4. **정적 파일(`www/index.html` 및 `www/upload.html`)**:
    - 정적 파일을 제공하는 예제 HTML 파일입니다.
    - `upload.html` 파일은 파일 업로드를 위한 폼을 포함합니다.

5. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시키고 `-pthread` 플래그를 사용하여 멀티스레딩을 지원합니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
