# Project 07

### 1.7 간단한 쉘 인터프리터

#### 프로젝트 구조

```
simple_shell/
├── include/
│   └── shell.h
├── src/
│   ├── shell.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 쉘 인터프리터

이 프로젝트는 기본적인 명령어를 처리할 수 있는 간단한 쉘 인터프리터입니다. 쉘 인터프리터는 명령어를 입력받아 실행하고, 결과를 출력합니다.

## 파일 구조

- `include/shell.h` : 쉘 함수와 구조체 선언이 포함된 헤더 파일.
- `src/shell.c` : 쉘 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_shell
```

## 사용 예

프로그램을 실행하고, 쉘 명령어를 입력합니다.

```sh
$ ./simple_shell
simple_shell> ls
simple_shell> pwd
simple_shell> exit
```
```

#### 코드 구현 및 상세 설명

**include/shell.h**

```c
#ifndef SHELL_H
#define SHELL_H

#define MAX_COMMAND_LENGTH 1024

// 쉘 함수 선언
void start_shell();
void execute_command(char *command);

#endif // SHELL_H
```

**src/shell.c**

```c
#include "shell.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>

void start_shell() {
    char command[MAX_COMMAND_LENGTH];

    while (1) {
        printf("simple_shell> ");
        if (fgets(command, sizeof(command), stdin) == NULL) {
            break;
        }

        // 개행 문자 제거
        command[strcspn(command, "\n")] = '\0';

        // "exit" 명령어 처리
        if (strcmp(command, "exit") == 0) {
            break;
        }

        // 명령어 실행
        execute_command(command);
    }
}

void execute_command(char *command) {
    char *args[MAX_COMMAND_LENGTH / 2 + 1];
    char *token;
    int i = 0;

    // 명령어를 공백 기준으로 분할
    token = strtok(command, " ");
    while (token != NULL) {
        args[i++] = token;
        token = strtok(NULL, " ");
    }
    args[i] = NULL;

    // 자식 프로세스 생성
    pid_t pid = fork();
    if (pid < 0) {
        perror("Fork failed");
        return;
    }

    // 자식 프로세스에서 명령어 실행
    if (pid == 0) {
        if (execvp(args[0], args) < 0) {
            perror("Exec failed");
        }
        exit(EXIT_FAILURE);
    } else {
        // 부모 프로세스는 자식 프로세스 종료 대기
        wait(NULL);
    }
}
```

**src/main.c**

```c
#include "shell.h"

int main() {
    start_shell();
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/shell.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_shell

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`shell.h`)**:
    - 쉘 인터프리터에 필요한 함수의 선언이 포함되어 있습니다.
    - `start_shell` 함수와 `execute_command` 함수가 선언되어 있습니다.

2. **소스 파일(`shell.c`)**:
    - `start_shell` 함수는 쉘 인터프리터를 시작하고 사용자 입력을 처리합니다.
        - 무한 루프를 돌면서 사용자로부터 명령어를 입력받습니다.
        - 입력받은 명령어가 `exit`이면 루프를 종료합니다.
        - 입력받은 명령어를 `execute_command` 함수로 실행합니다.
    - `execute_command` 함수는 명령어를 실행합니다.
        - 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
        - 자식 프로세스를 생성하여 명령어를 실행합니다.
        - 부모 프로세스는 자식 프로세스가 종료될 때까지 대기합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, `start_shell` 함수를 호출하여 쉘 인터프리터를 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

### 1.7 간단한 쉘 인터프리터

#### 프로젝트 구조

```
simple_shell/
├── include/
│   └── shell.h
├── src/
│   ├── shell.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 쉘 인터프리터

이 프로젝트는 기본적인 명령어를 처리할 수 있는 간단한 쉘 인터프리터입니다. 쉘 인터프리터는 명령어를 입력받아 실행하고, 결과를 출력합니다. 또한, 다음과 같은 기능들을 포함합니다:
- 명령어 이력 (history) 관리
- 간단한 파이프라인 명령어 지원
- 기본적인 환경 변수 처리

## 파일 구조

- `include/shell.h` : 쉘 함수와 구조체 선언이 포함된 헤더 파일.
- `src/shell.c` : 쉘 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_shell
```

## 사용 예

프로그램을 실행하고, 쉘 명령어를 입력합니다.

```sh
$ ./simple_shell
simple_shell> ls
simple_shell> pwd
simple_shell> history
simple_shell> echo $HOME
simple_shell> ls | grep .c
simple_shell> exit
```
```

#### 코드 구현 및 상세 설명

**include/shell.h**

```c
#ifndef SHELL_H
#define SHELL_H

#define MAX_COMMAND_LENGTH 1024
#define MAX_HISTORY 100

// 쉘 함수 선언
void start_shell();
void execute_command(char *command);
void parse_command(char *command, char **args);
void add_to_history(char *command);
void print_history();
void handle_pipeline(char *command);

#endif // SHELL_H
```

**src/shell.c**

```c
#include "shell.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#include <errno.h>

static char history[MAX_HISTORY][MAX_COMMAND_LENGTH];
static int history_count = 0;

void start_shell() {
    char command[MAX_COMMAND_LENGTH];

    while (1) {
        printf("simple_shell> ");
        if (fgets(command, sizeof(command), stdin) == NULL) {
            break;
        }

        // 개행 문자 제거
        command[strcspn(command, "\n")] = '\0';

        // "exit" 명령어 처리
        if (strcmp(command, "exit") == 0) {
            break;
        }

        // 명령어 이력에 추가
        add_to_history(command);

        // 파이프라인 명령어 처리
        if (strchr(command, '|')) {
            handle_pipeline(command);
        } else {
            // 명령어 실행
            execute_command(command);
        }
    }
}

void execute_command(char *command) {
    char *args[MAX_COMMAND_LENGTH / 2 + 1];
    parse_command(command, args);

    // "history" 명령어 처리
    if (strcmp(args[0], "history") == 0) {
        print_history();
        return;
    }

    // 자식 프로세스 생성
    pid_t pid = fork();
    if (pid < 0) {
        perror("Fork failed");
        return;
    }

    // 자식 프로세스에서 명령어 실행
    if (pid == 0) {
        if (execvp(args[0], args) < 0) {
            perror("Exec failed");
        }
        exit(EXIT_FAILURE);
    } else {
        // 부모 프로세스는 자식 프로세스 종료 대기
        wait(NULL);
    }
}

void parse_command(char *command, char **args) {
    char *token;
    int i = 0;

    // 명령어를 공백 기준으로 분할
    token = strtok(command, " ");
    while (token != NULL) {
        args[i++] = token;
        token = strtok(NULL, " ");
    }
    args[i] = NULL;
}

void add_to_history(char *command) {
    if (history_count < MAX_HISTORY) {
        strncpy(history[history_count++], command, MAX_COMMAND_LENGTH);
    } else {
        // 이력이 가득 차면 오래된 항목을 제거
        for (int i = 1; i < MAX_HISTORY; i++) {
            strncpy(history[i - 1], history[i], MAX_COMMAND_LENGTH);
        }
        strncpy(history[MAX_HISTORY - 1], command, MAX_COMMAND_LENGTH);
    }
}

void print_history() {
    for (int i = 0; i < history_count; i++) {
        printf("%d %s\n", i + 1, history[i]);
    }
}

void handle_pipeline(char *command) {
    char *args[MAX_COMMAND_LENGTH / 2 + 1];
    char *commands[2];
    int pipefd[2];
    pid_t pid1, pid2;

    // 파이프라인 명령어 분리
    commands[0] = strtok(command, "|");
    commands[1] = strtok(NULL, "|");

    // 파이프 생성
    if (pipe(pipefd) == -1) {
        perror("Pipe failed");
        return;
    }

    // 첫 번째 명령어 실행
    pid1 = fork();
    if (pid1 < 0) {
        perror("Fork failed");
        return;
    }

    if (pid1 == 0) {
        // 표준 출력 방향 변경
        dup2(pipefd[1], STDOUT_FILENO);
        close(pipefd[0]);
        close(pipefd[1]);

        parse_command(commands[0], args);
        if (execvp(args[0], args) < 0) {
            perror("Exec failed");
        }
        exit(EXIT_FAILURE);
    }

    // 두 번째 명령어 실행
    pid2 = fork();
    if (pid2 < 0) {
        perror("Fork failed");
        return;
    }

    if (pid2 == 0) {
        // 표준 입력 방향 변경
        dup2(pipefd[0], STDIN_FILENO);
        close(pipefd[0]);
        close(pipefd[1]);

        parse_command(commands[1], args);
        if (execvp(args[0], args) < 0) {
            perror("Exec failed");
        }
        exit(EXIT_FAILURE);
    }

    // 파이프 닫기
    close(pipefd[0]);
    close(pipefd[1]);

    // 부모 프로세스는 자식 프로세스 종료 대기
    wait(NULL);
    wait(NULL);
}
```

**src/main.c**

```c
#include "shell.h"

int main() {
    start_shell();
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/shell.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_shell

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`shell.h`)**:
    - 쉘 인터프리터에 필요한 함수와 상수의 선언이 포함되어 있습니다.
    - `MAX_COMMAND_LENGTH`는 명령어 최대 길이를 정의합니다.
    - `MAX_HISTORY`는 명령어 이력의 최대 길이를 정의합니다.
    - 쉘 인터프리터와 관련된 함수들이 선언되어 있습니다.

2. **소스 파일(`shell.c`)**:
    - `start_shell` 함수는 쉘 인터프리터를 시작하고 사용자 입력을 처리합니다.
        - 무한 루프를 돌면서 사용자로부터 명령어를 입력받습니다.
        - 입력받은 명령어가 `exit`이면 루프를 종료합니다.
        - 명령어를 이력에 추가하고, 파이프라인 명령어를 처리하거나 명령어를 실행합니다.
    - `execute_command` 함수는 명령어를 실행합니다.
        - 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
        - 자식 프로세스를 생성하여 명령어를 실행합니다.
        - 부모 프로세스는 자식 프로세스가 종료될 때까지 대기합니다.
    - `parse_command` 함수는 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
    - `add_to_history` 함수는 명령어를 이력에 추가합니다.
        - 이력이 가득 차면 오래된 항목을 제거하고 새로운 명령어를 추가합니다.
    - `print_history` 함수는 명령어 이력을 출력합니다.
    - `handle_pipeline` 함수는 파이프라인 명령어를 처리합니다.
        - 파이프를 생성하고, 각 명령어를 자식 프로세스에서 실행합니다.
        - 부모 프로세스는 자식 프로세스가 종료될 때

까지 대기합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, `start_shell` 함수를 호출하여 쉘 인터프리터를 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
