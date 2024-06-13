# Project 11

## 10.11 모듈화된 대규모 프로젝트: 블로그 웹 애플리케이션

이 프로젝트는 다음과 같은 모듈로 구성됩니다
1. **HTTP 서버 모듈**
2. **라우팅 모듈**
3. **데이터베이스 모듈**
4. **HTML 템플릿 모듈**
5. **유틸리티 모듈**

### 프로젝트 구조

```
blog_web_app/
├── include/
│   ├── http_server.h
│   ├── router.h
│   ├── database.h
│   ├── template.h
│   └── utils.h
├── src/
│   ├── http_server.c
│   ├── router.c
│   ├── database.c
│   ├── template.c
│   ├── utils.c
│   └── main.c
├── templates/
│   ├── index.html
│   ├── post.html
│   └── new_post.html
├── posts/
│   └── (blog posts data)
├── Makefile
└── README.md
```

#### README.md

```markdown
# 블로그 웹 애플리케이션

이 프로젝트는 간단한 블로그 웹 애플리케이션입니다. 사용자는 블로그 글을 작성하고, 조회할 수 있습니다.

## 파일 구조

- `include/` : 헤더 파일들이 포함된 디렉토리
- `src/` : 소스 파일들이 포함된 디렉토리
- `templates/` : HTML 템플릿 파일들이 포함된 디렉토리
- `posts/` : 블로그 글 데이터가 저장되는 디렉토리
- `Makefile` : 프로젝트 빌드를 위한 Makefile
- `README.md` : 프로젝트 설명 파일

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./blog_web_app <port>
```

## 사용 예

프로그램을 실행하고, 웹 브라우저에서 `http://localhost:<port>`에 접속합니다.

```sh
$ ./blog_web_app 8080
Server is running on port 8080
```
```

### 코드 구현 및 상세 설명

**include/http_server.h**

```c
#ifndef HTTP_SERVER_H
#define HTTP_SERVER_H

#define BUFFER_SIZE 1024

void start_server(int port);
void handle_client(int client_socket);

#endif // HTTP_SERVER_H
```

**include/router.h**

```c
#ifndef ROUTER_H
#define ROUTER_H

void route_request(const char *method, const char *path, int client_socket);

#endif // ROUTER_H
```

**include/database.h**

```c
#ifndef DATABASE_H
#define DATABASE_H

typedef struct {
    char title[256];
    char content[4096];
} Post;

void init_database();
void save_post(const Post *post);
Post* load_posts(int *count);

#endif // DATABASE_H
```

**include/template.h**

```c
#ifndef TEMPLATE_H
#define TEMPLATE_H

void render_template(int client_socket, const char *template_name, const char *content);

#endif // TEMPLATE_H
```

**include/utils.h**

```c
#ifndef UTILS_H
#define UTILS_H

void send_response(int client_socket, int status_code, const char *status_text, const char *content_type, const char *body);
void log_request(const char *method, const char *path, int status);

#endif // UTILS_H
```

**src/http_server.c**

```c
#include "http_server.h"
#include "router.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <pthread.h>

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

    // 요청 라우팅
    route_request(method, path, client_socket);

    close(client_socket);
}
```

**src/router.c**

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            // 메인 페이지
            int post_count;
            Post *posts = load_posts(&post_count);
            char content[8192] = "<h1>Blog Posts</h1>";
            for (int i = 0; i < post_count; i++) {
                strcat(content, "<h2>");
                strcat(content, posts[i].title);
                strcat(content, "</h2><p>");
                strcat(content, posts[i].content);
                strcat(content, "</p>");
            }
            render_template(client_socket, "index.html", content);
        } else if (strcmp(path, "/new") == 0) {
            // 새 글 작성 페이지
            render_template(client_socket, "new_post.html", NULL);
        } else {
            // 정적 파일
            char full_path[512];
            snprintf(full_path, sizeof(full_path), "www%s", path);
            FILE *file = fopen(full_path, "r");
            if (file) {
                fseek(file, 0, SEEK_END);
                long length = ftell(file);
                fseek(file, 0, SEEK_SET);
                char *content = malloc(length + 1);
                fread(content, 1, length, file);
                content[length] = '\0';
                fclose(file);
                send_response(client_socket, 200, "OK", "text/html", content);
                free(content);
            } else {
                send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
            }
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            // 블로그 글 저장
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096];
            sscanf(buffer, "title=%255[^&]&content=%4095s", title, content);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Post saved</h1>");
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
    }
}
```

**src/database.c**

```c
#include "database.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define POSTS_DIR "posts"

void init_database() {
    mkdir(POSTS_DIR, 0755);
}

void save_post(const Post *post) {
    static int post_id = 1;
    char filename[512];


    snprintf(filename, sizeof(filename), "%s/post%d.txt", POSTS_DIR, post_id++);
    FILE *file = fopen(filename, "w");
    if (file) {
        fprintf(file, "%s\n%s", post->title, post->content);
        fclose(file);
    }
}

Post* load_posts(int *count) {
    *count = 0;
    static Post posts[100];
    char filename[512];
    for (int i = 1; ; i++) {
        snprintf(filename, sizeof(filename), "%s/post%d.txt", POSTS_DIR, i);
        FILE *file = fopen(filename, "r");
        if (!file) {
            break;
        }
        fgets(posts[*count].title, sizeof(posts[*count].title), file);
        fgets(posts[*count].content, sizeof(posts[*count].content), file);
        fclose(file);
        (*count)++;
    }
    return posts;
}
```

**src/template.c**

```c
#include "template.h"
#include "utils.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void render_template(int client_socket, const char *template_name, const char *content) {
    char template_path[512];
    snprintf(template_path, sizeof(template_path), "templates/%s", template_name);
    FILE *file = fopen(template_path, "r");
    if (file) {
        fseek(file, 0, SEEK_END);
        long length = ftell(file);
        fseek(file, 0, SEEK_SET);
        char *template_content = malloc(length + 1);
        fread(template_content, 1, length, file);
        template_content[length] = '\0';
        fclose(file);

        if (content) {
            char *placeholder = strstr(template_content, "{{content}}");
            if (placeholder) {
                long offset = placeholder - template_content;
                template_content[offset] = '\0';
                strcat(template_content, content);
                strcat(template_content, placeholder + 11);
            }
        }

        send_response(client_socket, 200, "OK", "text/html", template_content);
        free(template_content);
    } else {
        send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
    }
}
```

**src/utils.c**

```c
#include "utils.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/socket.h>

void send_response(int client_socket, int status_code, const char *status_text, const char *content_type, const char *body) {
    char header[BUFFER_SIZE];
    snprintf(header, sizeof(header),
             "HTTP/1.1 %d %s\r\n"
             "Content-Type: %s\r\n"
             "\r\n",
             status_code, status_text, content_type);
    write(client_socket, header, strlen(header));
    write(client_socket, body, strlen(body));
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
#include "http_server.h"
#include "database.h"
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <port>\n", argv[0]);
        return 1;
    }

    int port = atoi(argv[1]);
    init_database();
    start_server(port);

    return 0;
}
```

**templates/index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Blog</title>
</head>
<body>
    <h1>Welcome to the Blog!</h1>
    {{content}}
    <a href="/new">Write a new post</a>
</body>
</html>
```

**templates/new_post.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>New Post</title>
</head>
<body>
    <h1>Write a New Post</h1>
    <form action="/post" method="post">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title"><br>
        <label for="content">Content:</label><br>
        <textarea id="content" name="content" rows="4" cols="50"></textarea><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall -pthread

SRCS = src/main.c src/http_server.c src/router.c src/database.c src/template.c src/utils.c
OBJS = $(SRCS:.c=.o)

TARGET = blog_web_app

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS) -pthread

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일**:
    - `http_server.h`, `router.h`, `database.h`, `template.h`, `utils.h` 각각의 모듈에 필요한 함수와 상수들이 선언되어 있습니다.

2. **소스 파일**:
    - `http_server.c`는 HTTP 서버를 구현합니다.
        - `start_server` 함수는 서버를 시작하고 클라이언트 요청을 처리합니다.
        - `handle_client` 함수는 클라이언트 요청을 읽고, HTTP 요청을 파싱하여 처리합니다.
    - `router.c`는 라우팅을 구현합니다.
        - `route_request` 함수는 HTTP 요청을 처리하고 적절한 응답을 반환합니다.
        - GET 요청에 대해 정적 파일을 제공하고, POST 요청에 대해 블로그 글을 저장합니다.
    - `database.c`는 데이터베이스 기능을 구현합니다.
        - `init_database` 함수는 데이터베이스를 초기화합니다.
        - `save_post` 함수는 블로그 글을 저장합니다.
        - `load_posts` 함수는 블로그 글을 로드합니다.
    - `template.c`는 HTML 템플릿 렌더링을 구현합니다.
        - `render_template` 함수는 템플릿 파일을 읽고, 내용을 삽입하여 렌더링합니다.
    - `utils.c`는 유틸리티 함수들을 구현합니다.
        - `send_response` 함수는 클라이언트에 HTTP 응답을 보냅니다.
        - `log_request` 함수는 요청을 로그 파일에 기록합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 포트 번호를 받습니다.
    - `init_database` 함수를 호출하여 데이터베이스를 초기화하고, `start_server` 함수를 호출하여 웹 서버를 시작합니다.

4. **템플릿 파일**:
    - `index.html`과 `new_post.html` 템플릿 파일은 HTML 템플릿을 정의합니다.

5. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시키고 `-pthread` 플래그를 사용하여 멀티스레딩을 지원합니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

## 추가 기능
1. **사용자 인증**: 사용자 등록, 로그인, 로그아웃 기능 추가
2. **댓글 기능**: 블로그 포스트에 댓글 추가
3. **페이지네이션**: 블로그 포스트 목록 페이지네이션
4. **검색 기능**: 블로그 포스트 검색 기능

### 프로젝트 구조

```
blog_web_app/
├── include/
│   ├── http_server.h
│   ├── router.h
│   ├── database.h
│   ├── template.h
│   ├── utils.h
│   ├── auth.h
│   ├── comments.h
│   ├── pagination.h
│   └── search.h
├── src/
│   ├── http_server.c
│   ├── router.c
│   ├── database.c
│   ├── template.c
│   ├── utils.c
│   ├── auth.c
│   ├── comments.c
│   ├── pagination.c
│   ├── search.c
│   └── main.c
├── templates/
│   ├── index.html
│   ├── post.html
│   ├── new_post.html
│   ├── register.html
│   ├── login.html
│   ├── comment_form.html
│   └── search_results.html
├── posts/
│   └── (blog posts data)
├── users/
│   └── (user data)
├── comments/
│   └── (comments data)
├── Makefile
└── README.md
```

### 코드 구현 및 상세 설명

**include/auth.h**

```c
#ifndef AUTH_H
#define AUTH_H

typedef struct {
    char username[50];
    char password[50];
} User;

void init_users();
int register_user(const char *username, const char *password);
int login_user(const char *username, const char *password);
void logout_user();

#endif // AUTH_H
```

**include/comments.h**

```c
#ifndef COMMENTS_H
#define COMMENTS_H

typedef struct {
    char username[50];
    char comment[256];
} Comment;

void add_comment(const char *post_id, const Comment *comment);
Comment* load_comments(const char *post_id, int *count);

#endif // COMMENTS_H
```

**include/pagination.h**

```c
#ifndef PAGINATION_H
#define PAGINATION_H

void paginate_posts(int client_socket, int page, int posts_per_page);

#endif // PAGINATION_H
```

**include/search.h**

```c
#ifndef SEARCH_H
#define SEARCH_H

void search_posts(int client_socket, const char *query);

#endif // SEARCH_H
```

**src/auth.c**

```c
#include "auth.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define USERS_DIR "users"

void init_users() {
    mkdir(USERS_DIR, 0755);
}

int register_user(const char *username, const char *password) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", USERS_DIR, username);
    FILE *file = fopen(filename, "r");
    if (file) {
        fclose(file);
        return 0; // 이미 존재하는 사용자
    }
    file = fopen(filename, "w");
    if (!file) {
        return 0;
    }
    fprintf(file, "%s\n%s", username, password);
    fclose(file);
    return 1;
}

int login_user(const char *username, const char *password) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", USERS_DIR, username);
    FILE *file = fopen(filename, "r");
    if (!file) {
        return 0;
    }
    char stored_username[50], stored_password[50];
    fscanf(file, "%s\n%s", stored_username, stored_password);
    fclose(file);
    if (strcmp(username, stored_username) == 0 && strcmp(password, stored_password) == 0) {
        // 세션을 생성하거나 사용자 상태를 업데이트할 수 있습니다.
        return 1;
    }
    return 0;
}

void logout_user() {
    // 세션을 삭제하거나 사용자 상태를 업데이트할 수 있습니다.
}
```

**src/comments.c**

```c
#include "comments.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define COMMENTS_DIR "comments"

void add_comment(const char *post_id, const Comment *comment) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", COMMENTS_DIR, post_id);
    FILE *file = fopen(filename, "a");
    if (!file) {
        mkdir(COMMENTS_DIR, 0755);
        file = fopen(filename, "a");
    }
    fprintf(file, "%s: %s\n", comment->username, comment->comment);
    fclose(file);
}

Comment* load_comments(const char *post_id, int *count) {
    static Comment comments[100];
    *count = 0;
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", COMMENTS_DIR, post_id);
    FILE *file = fopen(filename, "r");
    if (!file) {
        return comments;
    }
    while (fscanf(file, "%49[^:]: %255[^\n]\n", comments[*count].username, comments[*count].comment) == 2) {
        (*count)++;
    }
    fclose(file);
    return comments;
}
```

**src/pagination.c**

```c
#include "pagination.h"
#include "database.h"
#include "template.h"
#include "utils.h"
#include <stdio.h>
#include <string.h>

void paginate_posts(int client_socket, int page, int posts_per_page) {
    int post_count;
    Post *posts = load_posts(&post_count);
    int start = (page - 1) * posts_per_page;
    int end = start + posts_per_page;

    if (start >= post_count) {
        send_response(client_socket, 404, "Not Found", "text/html", "<h1>No more posts</h1>");
        return;
    }

    char content[8192] = "<h1>Blog Posts</h1>";
    for (int i = start; i < end && i < post_count; i++) {
        strcat(content, "<h2>");
        strcat(content, posts[i].title);
        strcat(content, "</h2><p>");
        strcat(content, posts[i].content);
        strcat(content, "</p>");
    }
    render_template(client_socket, "index.html", content);
}
```

**src/search.c**

```c
#include "search.h"
#include "database.h"
#include "template.h"
#include "utils.h"
#include <stdio.h>
#include <string.h>

void search_posts(int client_socket, const char *query) {
    int post_count;
    Post *posts = load_posts(&post_count);

    char content[8192] = "<h1>Search Results</h1>";
    for (int i = 0; i < post_count; i++) {
        if (strstr(posts[i].title, query) || strstr(posts[i].content, query)) {
            strcat(content, "<h2>");
            strcat(content, posts[i].title);
            strcat(content, "</h2><p>");
            strcat(content, posts[i].content);
            strcat(content, "</p>");
        }
    }
    render_template(client_socket, "search_results.html", content);
}
```

**src/router.c**

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "auth.h"
#include "comments.h"
#include "pagination.h"
#include "search.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            paginate_posts(client_socket, 1, 5);
        } else if (strcmp(path, "/new") == 0) {
            render_template(client_socket, "new_post.html", NULL);
        } else if (strcmp(path, "/register") == 0) {
            render_template(client_socket, "register.html", NULL);
        } else if (strcmp(path, "/login") == 0) {
            render_template(client_socket, "login.html", NULL);
        } else if (strncmp(path, "/search?", 8) == 0) {
            char query[256];
            sscanf(path + 8, "q=%255s", query);
            search_posts(client_socket, query);
        } else if (strncmp(path, "/page/", 6) == 0) {
            int page;
            sscanf(path + 6, "%d", &page);
            paginate_posts(client_socket, page, 5);
        } else {
            char full_path[512];
            snprintf(full_path, sizeof(full_path), "www%s", path);
            FILE *file = fopen(full_path, "r");
            if (file) {
                fseek(file, 0, SEEK_END);
                long length = ftell(file);
                fseek(file, 0, SEEK_SET);
                char *content = malloc(length + 1);
                fread(content, 1, length,

 file);
                content[length] = '\0';
                fclose(file);
                send_response(client_socket, 200, "OK", "text/html", content);
                free(content);
            } else {
                send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
            }
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096];
            sscanf(buffer, "title=%255[^&]&content=%4095s", title, content);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Post saved</h1>");
        } else if (strcmp(path, "/register") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (register_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Registration successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Registration failed</h1>");
            }
        } else if (strcmp(path, "/login") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (login_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Login successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Login failed</h1>");
            }
        } else if (strncmp(path, "/comment/", 9) == 0) {
            char post_id[50];
            sscanf(path + 9, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], comment_text[256];
            sscanf(buffer, "username=%49[^&]&comment=%255s", username, comment_text);
            Comment comment;
            strncpy(comment.username, username, sizeof(comment.username));
            strncpy(comment.comment, comment_text, sizeof(comment.comment));
            add_comment(post_id, &comment);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Comment added</h1>");
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
    }
}
```

**templates/register.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Register</title>
</head>
<body>
    <h1>Register</h1>
    <form action="/register" method="post">
        <label for="username">Username:</label><br>
        <input type="text" id="username" name="username"><br>
        <label for="password">Password:</label><br>
        <input type="password" id="password" name="password"><br>
        <input type="submit" value="Register">
    </form>
</body>
</html>
```

**templates/login.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="/login" method="post">
        <label for="username">Username:</label><br>
        <input type="text" id="username" name="username"><br>
        <label for="password">Password:</label><br>
        <input type="password" id="password" name="password"><br>
        <input type="submit" value="Login">
    </form>
</body>
</html>
```

**templates/comment_form.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Add Comment</title>
</head>
<body>
    <h1>Add a Comment</h1>
    <form action="/comment/POST_ID" method="post">
        <label for="username">Username:</label><br>
        <input type="text" id="username" name="username"><br>
        <label for="comment">Comment:</label><br>
        <textarea id="comment" name="comment" rows="4" cols="50"></textarea><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

**templates/search_results.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Search Results</title>
</head>
<body>
    <h1>Search Results</h1>
    {{content}}
</body>
</html>
```

### 추가된 기능 설명

1. **사용자 인증**:
    - `register.html`, `login.html` 템플릿 파일과 `auth.h`, `auth.c` 파일을 추가하여 사용자 등록과 로그인을 처리합니다.
    - `register_user`, `login_user` 함수로 사용자 데이터를 저장하고 인증합니다.

2. **댓글 기능**:
    - `comment_form.html` 템플릿 파일과 `comments.h`, `comments.c` 파일을 추가하여 댓글을 작성하고 조회합니다.
    - `add_comment`, `load_comments` 함수로 댓글 데이터를 저장하고 로드합니다.

3. **페이지네이션**:
    - `pagination.h`, `pagination.c` 파일을 추가하여 블로그 포스트 목록을 페이지네이션합니다.
    - `paginate_posts` 함수로 페이지별로 포스트를 로드합니다.

4. **검색 기능**:
    - `search_results.html` 템플릿 파일과 `search.h`, `search.c` 파일을 추가하여 블로그 포스트를 검색합니다.
    - `search_posts` 함수로 검색어에 맞는 포스트를 찾습니다.

### 코드 설명

위에 추가된 파일과 함수들을 통해 블로그 웹 애플리케이션을 확장하여 사용자 인증, 댓글 기능, 페이지네이션, 검색 기능 등을 구현하였습니다. 

---

## 추가 기능 2

1. **태그 기능**: 포스트에 태그를 추가하고, 태그별로 포스트를 필터링하는 기능.
2. **좋아요 기능**: 포스트에 좋아요를 누를 수 있는 기능.
3. **프로필 페이지**: 사용자가 자신의 프로필을 볼 수 있는 페이지.
4. **포스트 수정 및 삭제**: 사용자가 자신의 포스트를 수정하고 삭제할 수 있는 기능.
5. **다크 모드 지원**: 다크 모드를 지원하여 사용자에게 다양한 테마를 제공.
6. **알림 기능**: 새로운 포스트나 댓글에 대한 알림 기능.
7. **이미지 업로드**: 포스트에 이미지를 첨부할 수 있는 기능.
8. **사용자 역할**: 관리자, 일반 사용자 등의 역할을 추가하여 권한을 관리.
9. **RESTful API**: 블로그 웹 애플리케이션의 데이터를 외부에서 접근할 수 있도록 API를 제공.
10. **비밀번호 암호화**: 사용자 비밀번호를 암호화하여 저장하는 기능.

이 중에서 일부 기능을 선택하여 구현해 보겠습니다. 이번에는 **태그 기능**과 **포스트 수정 및 삭제 기능**을 추가하겠습니다.

### 추가 기능 구현

#### 1. 태그 기능

**include/tags.h**

```c
#ifndef TAGS_H
#define TAGS_H

typedef struct {
    char name[50];
} Tag;

void add_tag(const char *post_id, const Tag *tag);
Tag* load_tags(const char *post_id, int *count);
Post* filter_posts_by_tag(const char *tag, int *count);

#endif // TAGS_H
```

**src/tags.c**

```c
#include "tags.h"
#include "database.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define TAGS_DIR "tags"

void add_tag(const char *post_id, const Tag *tag) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", TAGS_DIR, post_id);
    FILE *file = fopen(filename, "a");
    if (!file) {
        mkdir(TAGS_DIR, 0755);
        file = fopen(filename, "a");
    }
    fprintf(file, "%s\n", tag->name);
    fclose(file);
}

Tag* load_tags(const char *post_id, int *count) {
    static Tag tags[100];
    *count = 0;
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", TAGS_DIR, post_id);
    FILE *file = fopen(filename, "r");
    if (!file) {
        return tags;
    }
    while (fscanf(file, "%49s", tags[*count].name) == 1) {
        (*count)++;
    }
    fclose(file);
    return tags;
}

Post* filter_posts_by_tag(const char *tag, int *count) {
    static Post posts[100];
    *count = 0;
    char filename[512];
    for (int i = 1; ; i++) {
        snprintf(filename, sizeof(filename), "%s/post%d.txt", TAGS_DIR, i);
        FILE *file = fopen(filename, "r");
        if (!file) {
            break;
        }
        char line[50];
        while (fgets(line, sizeof(line), file)) {
            if (strstr(line, tag)) {
                posts[*count] = *load_posts(&i);
                (*count)++;
                break;
            }
        }
        fclose(file);
    }
    return posts;
}
```

**src/router.c** (업데이트)

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "auth.h"
#include "comments.h"
#include "pagination.h"
#include "search.h"
#include "tags.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            paginate_posts(client_socket, 1, 5);
        } else if (strcmp(path, "/new") == 0) {
            render_template(client_socket, "new_post.html", NULL);
        } else if (strcmp(path, "/register") == 0) {
            render_template(client_socket, "register.html", NULL);
        } else if (strcmp(path, "/login") == 0) {
            render_template(client_socket, "login.html", NULL);
        } else if (strncmp(path, "/search?", 8) == 0) {
            char query[256];
            sscanf(path + 8, "q=%255s", query);
            search_posts(client_socket, query);
        } else if (strncmp(path, "/tag?", 5) == 0) {
            char tag[50];
            sscanf(path + 5, "q=%49s", tag);
            int post_count;
            Post* posts = filter_posts_by_tag(tag, &post_count);
            char content[8192] = "<h1>Posts with tag: ";
            strcat(content, tag);
            strcat(content, "</h1>");
            for (int i = 0; i < post_count; i++) {
                strcat(content, "<h2>");
                strcat(content, posts[i].title);
                strcat(content, "</h2><p>");
                strcat(content, posts[i].content);
                strcat(content, "</p>");
            }
            render_template(client_socket, "search_results.html", content);
        } else if (strncmp(path, "/page/", 6) == 0) {
            int page;
            sscanf(path + 6, "%d", &page);
            paginate_posts(client_socket, page, 5);
        } else {
            char full_path[512];
            snprintf(full_path, sizeof(full_path), "www%s", path);
            FILE *file = fopen(full_path, "r");
            if (file) {
                fseek(file, 0, SEEK_END);
                long length = ftell(file);
                fseek(file, 0, SEEK_SET);
                char *content = malloc(length + 1);
                fread(content, 1, length, file);
                content[length] = '\0';
                fclose(file);
                send_response(client_socket, 200, "OK", "text/html", content);
                free(content);
            } else {
                send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
            }
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096], tags[256];
            sscanf(buffer, "title=%255[^&]&content=%4095[^&]&tags=%255s", title, content, tags);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);

            char tag[50];
            char *token = strtok(tags, ",");
            while (token != NULL) {
                Tag new_tag;
                strncpy(new_tag.name, token, sizeof(new_tag.name));
                add_tag(post.title, &new_tag);
                token = strtok(NULL, ",");
            }

            send_response(client_socket, 200, "OK", "text/html", "<h1>Post saved</h1>");
        } else if (strcmp(path, "/register") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (register_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Registration successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Registration failed</h1>");
            }
        } else if (strcmp(path, "/login") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (login_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Login successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Login failed</h1>");
            }
        } else if (strncmp(path, "/comment/", 9) == 0) {
            char post_id[50];
            sscanf(path + 9, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read]

 = '\0';
            char username[50], comment_text[256];
            sscanf(buffer, "username=%49[^&]&comment=%255s", username, comment_text);
            Comment comment;
            strncpy(comment.username, username, sizeof(comment.username));
            strncpy(comment.comment, comment_text, sizeof(comment.comment));
            add_comment(post_id, &comment);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Comment added</h1>");
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
    }
}
```

#### 2. 포스트 수정 및 삭제 기능

**include/posts.h**

```c
#ifndef POSTS_H
#define POSTS_H

void edit_post(const char *post_id, const Post *new_post);
void delete_post(const char *post_id);

#endif // POSTS_H
```

**src/posts.c**

```c
#include "posts.h"
#include "database.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void edit_post(const char *post_id, const Post *new_post) {
    char filename[512];
    snprintf(filename, sizeof(filename), "posts/%s.txt", post_id);
    FILE *file = fopen(filename, "w");
    if (file) {
        fprintf(file, "%s\n%s", new_post->title, new_post->content);
        fclose(file);
    }
}

void delete_post(const char *post_id) {
    char filename[512];
    snprintf(filename, sizeof(filename), "posts/%s.txt", post_id);
    remove(filename);
}
```

**src/router.c** (업데이트)

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "auth.h"
#include "comments.h"
#include "pagination.h"
#include "search.h"
#include "tags.h"
#include "posts.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        if (strcmp(path, "/") == 0) {
            paginate_posts(client_socket, 1, 5);
        } else if (strcmp(path, "/new") == 0) {
            render_template(client_socket, "new_post.html", NULL);
        } else if (strcmp(path, "/register") == 0) {
            render_template(client_socket, "register.html", NULL);
        } else if (strcmp(path, "/login") == 0) {
            render_template(client_socket, "login.html", NULL);
        } else if (strncmp(path, "/search?", 8) == 0) {
            char query[256];
            sscanf(path + 8, "q=%255s", query);
            search_posts(client_socket, query);
        } else if (strncmp(path, "/tag?", 5) == 0) {
            char tag[50];
            sscanf(path + 5, "q=%49s", tag);
            int post_count;
            Post* posts = filter_posts_by_tag(tag, &post_count);
            char content[8192] = "<h1>Posts with tag: ";
            strcat(content, tag);
            strcat(content, "</h1>");
            for (int i = 0; i < post_count; i++) {
                strcat(content, "<h2>");
                strcat(content, posts[i].title);
                strcat(content, "</h2><p>");
                strcat(content, posts[i].content);
                strcat(content, "</p>");
            }
            render_template(client_socket, "search_results.html", content);
        } else if (strncmp(path, "/page/", 6) == 0) {
            int page;
            sscanf(path + 6, "%d", &page);
            paginate_posts(client_socket, page, 5);
        } else if (strncmp(path, "/edit/", 6) == 0) {
            char post_id[50];
            sscanf(path + 6, "%49s", post_id);
            Post* post = load_posts(&post_id);
            if (post) {
                render_template(client_socket, "edit_post.html", post->content);
            } else {
                send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
            }
        } else {
            char full_path[512];
            snprintf(full_path, sizeof(full_path), "www%s", path);
            FILE *file = fopen(full_path, "r");
            if (file) {
                fseek(file, 0, SEEK_END);
                long length = ftell(file);
                fseek(file, 0, SEEK_SET);
                char *content = malloc(length + 1);
                fread(content, 1, length, file);
                content[length] = '\0';
                fclose(file);
                send_response(client_socket, 200, "OK", "text/html", content);
                free(content);
            } else {
                send_response(client_socket, 404, "Not Found", "text/html", "<h1>404 Not Found</h1>");
            }
        }
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096], tags[256];
            sscanf(buffer, "title=%255[^&]&content=%4095[^&]&tags=%255s", title, content, tags);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);

            char tag[50];
            char *token = strtok(tags, ",");
            while (token != NULL) {
                Tag new_tag;
                strncpy(new_tag.name, token, sizeof(new_tag.name));
                add_tag(post.title, &new_tag);
                token = strtok(NULL, ",");
            }

            send_response(client_socket, 200, "OK", "text/html", "<h1>Post saved</h1>");
        } else if (strcmp(path, "/register") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (register_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Registration successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Registration failed</h1>");
            }
        } else if (strcmp(path, "/login") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (login_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Login successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Login failed</h1>");
            }
        } else if (strncmp(path, "/comment/", 9) == 0) {
            char post_id[50];
            sscanf(path + 9, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], comment_text[256];
            sscanf(buffer, "username=%49[^&]&comment=%255s", username, comment_text);
            Comment comment;
            strncpy(comment.username, username, sizeof(comment.username));
            strncpy(comment.comment, comment_text, sizeof(comment.comment));
            add_comment(post_id, &comment);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Comment added</h1>");
        } else if (strncmp(path, "/edit/", 6) == 0) {
            char post_id[50];
            sscanf(path + 6, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096];
            sscanf(buffer, "title=%255[^&]&content=%4095s", title, content);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            edit_post(post_id, &post);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Post updated</h1>");
        } else if (strncmp(path, "/delete/", 8) == 0) {
            char post_id[50];
            sscanf(path + 8, "%49s", post_id);
            delete_post(post_id);
            send_response(client_socket

, 200, "OK", "text/html", "<h1>Post deleted</h1>");
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
    }
}
```

**templates/edit_post.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Edit Post</title>
</head>
<body>
    <h1>Edit Post</h1>
    <form action="/edit/POST_ID" method="post">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title" value="{{title}}"><br>
        <label for="content">Content:</label><br>
        <textarea id="content" name="content" rows="4" cols="50">{{content}}</textarea><br>
        <input type="submit" value="Update">
    </form>
</body>
</html>
```

### 추가된 기능 설명

1. **태그 기능**:
    - `tags.h`, `tags.c` 파일을 추가하여 태그를 포스트에 추가하고, 태그별로 포스트를 필터링하는 기능을 구현합니다.
    - `add_tag`, `load_tags`, `filter_posts_by_tag` 함수를 통해 태그를 관리합니다.

2. **포스트 수정 및 삭제 기능**:
    - `posts.h`, `posts.c` 파일을 추가하여 포스트를 수정하고 삭제하는 기능을 구현합니다.
    - `edit_post`, `delete_post` 함수를 통해 포스트를 관리합니다.
    - `edit_post.html` 템플릿 파일을 추가하여 포스트 수정 폼을 제공합니다.

---

## 추가 기능 3 
1. 이미지 업로드
2. 사용자 역할

### 추가 기능 구현

#### 1. 이미지 업로드 기능

**include/images.h**

```c
#ifndef IMAGES_H
#define IMAGES_H

void upload_image(const char *filename, const char *content);
char* get_image_path(const char *filename);

#endif // IMAGES_H
```

**src/images.c**

```c
#include "images.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define IMAGES_DIR "uploads/images"

void upload_image(const char *filename, const char *content) {
    char filepath[512];
    snprintf(filepath, sizeof(filepath), "%s/%s", IMAGES_DIR, filename);
    FILE *file = fopen(filepath, "wb");
    if (!file) {
        mkdir(IMAGES_DIR, 0755);
        file = fopen(filepath, "wb");
    }
    if (file) {
        fwrite(content, 1, strlen(content), file);
        fclose(file);
    }
}

char* get_image_path(const char *filename) {
    static char filepath[512];
    snprintf(filepath, sizeof(filepath), "%s/%s", IMAGES_DIR, filename);
    return filepath;
}
```

**src/router.c** (업데이트)

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "auth.h"
#include "comments.h"
#include "pagination.h"
#include "search.h"
#include "tags.h"
#include "posts.h"
#include "images.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        // 기존 GET 요청 처리
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096], tags[256], image_filename[256];
            sscanf(buffer, "title=%255[^&]&content=%4095[^&]&tags=%255[^&]&image=%255s", title, content, tags, image_filename);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);

            char tag[50];
            char *token = strtok(tags, ",");
            while (token != NULL) {
                Tag new_tag;
                strncpy(new_tag.name, token, sizeof(new_tag.name));
                add_tag(post.title, &new_tag);
                token = strtok(NULL, ",");
            }

            // 이미지 업로드 처리
            if (strlen(image_filename) > 0) {
                upload_image(image_filename, content);  // content should be image data
            }

            send_response(client_socket, 200, "OK", "text/html", "<h1>Post saved</h1>");
        } else if (strncmp(path, "/upload_image", 13) == 0) {
            // 이미지 업로드 처리
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char filename[256], image_content[4096];
            sscanf(buffer, "filename=%255[^&]&content=%4095s", filename, image_content);
            upload_image(filename, image_content);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Image uploaded</h1>");
        } else {
            // 기존 POST 요청 처리
        }
    } else {
        // 기존 다른 요청 처리
    }
}
```

**templates/new_post.html** (업데이트)

```html
<!DOCTYPE html>
<html>
<head>
    <title>New Post</title>
</head>
<body>
    <h1>Write a New Post</h1>
    <form action="/post" method="post" enctype="multipart/form-data">
        <label for="title">Title:</label><br>
        <input type="text" id="title" name="title"><br>
        <label for="content">Content:</label><br>
        <textarea id="content" name="content" rows="4" cols="50"></textarea><br>
        <label for="tags">Tags (comma-separated):</label><br>
        <input type="text" id="tags" name="tags"><br>
        <label for="image">Image:</label><br>
        <input type="file" id="image" name="image"><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

#### 2. 사용자 역할 기능

**include/auth.h** (업데이트)

```c
#ifndef AUTH_H
#define AUTH_H

typedef enum {
    ROLE_USER,
    ROLE_ADMIN
} UserRole;

typedef struct {
    char username[50];
    char password[50];
    UserRole role;
} User;

void init_users();
int register_user(const char *username, const char *password, UserRole role);
int login_user(const char *username, const char *password);
UserRole get_user_role(const char *username);
void logout_user();

#endif // AUTH_H
```

**src/auth.c** (업데이트)

```c
#include "auth.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define USERS_DIR "users"

void init_users() {
    mkdir(USERS_DIR, 0755);
}

int register_user(const char *username, const char *password, UserRole role) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", USERS_DIR, username);
    FILE *file = fopen(filename, "r");
    if (file) {
        fclose(file);
        return 0; // 이미 존재하는 사용자
    }
    file = fopen(filename, "w");
    if (!file) {
        return 0;
    }
    fprintf(file, "%s\n%s\n%d", username, password, role);
    fclose(file);
    return 1;
}

int login_user(const char *username, const char *password) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", USERS_DIR, username);
    FILE *file = fopen(filename, "r");
    if (!file) {
        return 0;
    }
    char stored_username[50], stored_password[50];
    int stored_role;
    fscanf(file, "%s\n%s\n%d", stored_username, stored_password, &stored_role);
    fclose(file);
    if (strcmp(username, stored_username) == 0 && strcmp(password, stored_password) == 0) {
        // 세션을 생성하거나 사용자 상태를 업데이트할 수 있습니다.
        return 1;
    }
    return 0;
}

UserRole get_user_role(const char *username) {
    char filename[512];
    snprintf(filename, sizeof(filename), "%s/%s.txt", USERS_DIR, username);
    FILE *file = fopen(filename, "r");
    if (!file) {
        return ROLE_USER; // 기본값으로 USER 반환
    }
    char stored_username[50], stored_password[50];
    int stored_role;
    fscanf(file, "%s\n%s\n%d", stored_username, stored_password, &stored_role);
    fclose(file);
    return (UserRole)stored_role;
}

void logout_user() {
    // 세션을 삭제하거나 사용자 상태를 업데이트할 수 있습니다.
}
```

**src/router.c** (업데이트)

```c
#include "router.h"
#include "template.h"
#include "database.h"
#include "auth.h"
#include "comments.h"
#include "pagination.h"
#include "search.h"
#include "tags.h"
#include "posts.h"
#include "images.h"
#include "utils.h"
#include <string.h>
#include <stdio.h>

void route_request(const char *method, const char *path, int client_socket) {
    if (strcmp(method, "GET") == 0) {
        // 기존 GET 요청 처리
    } else if (strcmp(method, "POST") == 0) {
        if (strcmp(path, "/post") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096], tags[256], image_filename[256];
            sscanf(buffer, "title=%255[^&]&content=%4095[^&]&tags=%255[^&]&image=%255s", title, content, tags, image_filename);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            save_post(&post);

            char tag[50];
            char *token = strtok(tags, ",");
            while (token != NULL) {
                Tag new_tag;
                strncpy(new_tag.name, token, sizeof(new_tag.name));
                add_tag(post.title, &new_tag);
                token = strtok(NULL, ",");
            }

            // 이미지 업로드 처리
            if (strlen(image_filename) > 0) {
                upload_image(image_filename, content);  // content should be image data
            }

            send_response(client_socket, 200, "OK", "text/html", "<h

1>Post saved</h1>");
        } else if (strncmp(path, "/upload_image", 13) == 0) {
            // 이미지 업로드 처리
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char filename[256], image_content[4096];
            sscanf(buffer, "filename=%255[^&]&content=%4095s", filename, image_content);
            upload_image(filename, image_content);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Image uploaded</h1>");
        } else if (strcmp(path, "/register") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (register_user(username, password, ROLE_USER)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Registration successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Registration failed</h1>");
            }
        } else if (strcmp(path, "/register_admin") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (register_user(username, password, ROLE_ADMIN)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Admin registration successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Admin registration failed</h1>");
            }
        } else if (strcmp(path, "/login") == 0) {
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], password[50];
            sscanf(buffer, "username=%49[^&]&password=%49s", username, password);
            if (login_user(username, password)) {
                send_response(client_socket, 200, "OK", "text/html", "<h1>Login successful</h1>");
            } else {
                send_response(client_socket, 400, "Bad Request", "text/html", "<h1>Login failed</h1>");
            }
        } else if (strncmp(path, "/comment/", 9) == 0) {
            char post_id[50];
            sscanf(path + 9, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char username[50], comment_text[256];
            sscanf(buffer, "username=%49[^&]&comment=%255s", username, comment_text);
            Comment comment;
            strncpy(comment.username, username, sizeof(comment.username));
            strncpy(comment.comment, comment_text, sizeof(comment.comment));
            add_comment(post_id, &comment);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Comment added</h1>");
        } else if (strncmp(path, "/edit/", 6) == 0) {
            char post_id[50];
            sscanf(path + 6, "%49s", post_id);
            char buffer[4096];
            int bytes_read = read(client_socket, buffer, sizeof(buffer) - 1);
            buffer[bytes_read] = '\0';
            char title[256], content[4096];
            sscanf(buffer, "title=%255[^&]&content=%4095s", title, content);
            Post post;
            strncpy(post.title, title, sizeof(post.title));
            strncpy(post.content, content, sizeof(post.content));
            edit_post(post_id, &post);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Post updated</h1>");
        } else if (strncmp(path, "/delete/", 8) == 0) {
            char post_id[50];
            sscanf(path + 8, "%49s", post_id);
            delete_post(post_id);
            send_response(client_socket, 200, "OK", "text/html", "<h1>Post deleted</h1>");
        } else {
            send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
        }
    } else {
        send_response(client_socket, 501, "Not Implemented", "text/html", "<h1>501 Not Implemented</h1>");
    }
}
```

**templates/register_admin.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Admin Registration</title>
</head>
<body>
    <h1>Admin Registration</h1>
    <form action="/register_admin" method="post">
        <label for="username">Username:</label><br>
        <input type="text" id="username" name="username"><br>
        <label for="password">Password:</label><br>
        <input type="password" id="password" name="password"><br>
        <input type="submit" value="Register">
    </form>
</body>
</html>
```

### 추가된 기능 설명

1. **이미지 업로드 기능**:
    - `images.h`, `images.c` 파일을 추가하여 이미지 업로드 기능을 구현합니다.
    - `upload_image` 함수로 이미지를 저장하고, `get_image_path` 함수로 이미지를 로드합니다.
    - `new_post.html` 템플릿 파일을 업데이트하여 이미지 업로드 폼을 추가합니다.

2. **사용자 역할 기능**:
    - `auth.h`, `auth.c` 파일을 업데이트하여 사용자 역할(ROLE_USER, ROLE_ADMIN)을 추가합니다.
    - `register_user` 함수에 역할을 추가하여 사용자 등록 시 역할을 설정할 수 있습니다.
    - `get_user_role` 함수로 사용자의 역할을 가져올 수 있습니다.
    - `register_admin.html` 템플릿 파일을 추가하여 관리자 등록 폼을 제공합니다.

---

### 마무리

1. **기본 웹 서버 기능**: HTTP 요청 처리, 정적 파일 제공
2. **사용자 인증**: 사용자 등록, 로그인, 로그아웃, 사용자 역할 관리
3. **댓글 기능**: 블로그 포스트에 댓글 작성 및 조회
4. **페이지네이션**: 블로그 포스트 목록 페이지네이션
5. **검색 기능**: 블로그 포스트 검색
6. **태그 기능**: 포스트에 태그 추가 및 태그별 필터링
7. **포스트 수정 및 삭제 기능**: 사용자가 자신의 포스트를 수정하고 삭제할 수 있는 기능
8. **이미지 업로드**: 포스트에 이미지 첨부

이러한 기능들을 통해 사용자들은 블로그 웹 애플리케이션을 사용하여 블로그 글을 작성하고, 조회하며, 댓글을 남길 수 있습니다. 또한, 이미지 업로드와 태그 기능을 통해 더욱 풍부한 콘텐츠를 제공할 수 있습니다.

### 최종 파일 구조

```
blog_web_app/
├── include/
│   ├── auth.h
│   ├── comments.h
│   ├── database.h
│   ├── http_server.h
│   ├── images.h
│   ├── pagination.h
│   ├── posts.h
│   ├── router.h
│   ├── search.h
│   ├── tags.h
│   ├── template.h
│   └── utils.h
├── src/
│   ├── auth.c
│   ├── comments.c
│   ├── database.c
│   ├── http_server.c
│   ├── images.c
│   ├── main.c
│   ├── pagination.c
│   ├── posts.c
│   ├── router.c
│   ├── search.c
│   ├── tags.c
│   ├── template.c
│   └── utils.c
├── templates/
│   ├── comment_form.html
│   ├── edit_post.html
│   ├── index.html
│   ├── login.html
│   ├── new_post.html
│   ├── register.html
│   ├── register_admin.html
│   └── search_results.html
├── uploads/
│   └── images/
├── users/
│   └── (user data)
├── comments/
│   └── (comments data)
├── posts/
│   └── (blog posts data)
├── Makefile
└── README.md
```

### README.md (최종)

```markdown
# 블로그 웹 애플리케이션

이 프로젝트는 기본적인 HTTP 요청을 처리하는 블로그 웹 애플리케이션입니다. 주요 기능으로는 사용자 인증, 댓글 기능, 페이지네이션, 검색, 태그, 이미지 업로드, 포스트 수정 및 삭제 기능이 있습니다.

## 파일 구조

- `include/` : 헤더 파일들이 포함된 디렉토리
- `src/` : 소스 파일들이 포함된 디렉토리
- `templates/` : HTML 템플릿 파일들이 포함된 디렉토리
- `uploads/` : 업로드된 이미지 파일들이 저장되는 디렉토리
- `users/` : 사용자 데이터가 저장되는 디렉토리
- `comments/` : 댓글 데이터가 저장되는 디렉토리
- `posts/` : 블로그 글 데이터가 저장되는 디렉토리
- `Makefile` : 프로젝트 빌드를 위한 Makefile
- `README.md` : 프로젝트 설명 파일

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./blog_web_app <port>
```

## 사용 예

프로그램을 실행하고, 웹 브라우저에서 `http://localhost:<port>`에 접속합니다.

```sh
$ ./blog_web_app 8080
Server is running on port 8080
```
```
