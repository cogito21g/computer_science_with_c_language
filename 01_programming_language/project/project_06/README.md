# Project 06

### 1.6 텍스트 에디터

#### 프로젝트 구조

```
simple_text_editor/
├── include/
│   └── editor.h
├── src/
│   ├── editor.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 텍스트 에디터

이 프로젝트는 기본적인 텍스트 편집 기능을 제공하는 간단한 텍스트 에디터입니다. 사용자는 파일을 열고, 텍스트를 입력하고, 파일을 저장할 수 있습니다.

## 파일 구조

- `include/editor.h` : 에디터 함수와 구조체 선언이 포함된 헤더 파일.
- `src/editor.c` : 에디터 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_text_editor <filename>
```

## 사용 예

프로그램을 실행하고, 텍스트를 입력하고, `Ctrl+S`로 파일을 저장하고, `Ctrl+Q`로 에디터를 종료합니다.

```sh
$ ./simple_text_editor sample.txt
```
```

#### 코드 구현 및 상세 설명

**include/editor.h**

```c
#ifndef EDITOR_H
#define EDITOR_H

#define CTRL_KEY(k) ((k) & 0x1f)

void enable_raw_mode();
void disable_raw_mode();
void editor_process_keypress();

#endif // EDITOR_H
```

**src/editor.c**

```c
#include "editor.h"
#include <unistd.h>
#include <termios.h>
#include <stdlib.h>
#include <stdio.h>

// 터미널 초기 설정 저장
struct termios orig_termios;

// RAW 모드 설정 해제
void disable_raw_mode() {
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios);
}

// RAW 모드 설정
void enable_raw_mode() {
    tcgetattr(STDIN_FILENO, &orig_termios);
    atexit(disable_raw_mode);

    struct termios raw = orig_termios;
    raw.c_lflag &= ~(ECHO | ICANON | ISIG | IEXTEN);
    raw.c_iflag &= ~(IXON | BRKINT | INPCK | ISTRIP | ICRNL);
    raw.c_oflag &= ~(OPOST);
    raw.c_cflag |= (CS8);
    raw.c_cc[VMIN] = 0;
    raw.c_cc[VTIME] = 1;

    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

// 키 입력 처리
void editor_process_keypress() {
    char c;
    while (1) {
        if (read(STDIN_FILENO, &c, 1) == -1 && errno != EAGAIN) {
            perror("read");
            exit(EXIT_FAILURE);
        }

        if (c == CTRL_KEY('q')) {
            break;
        }
        
        if (c == CTRL_KEY('s')) {
            printf("\x1b[H\x1b[J");
            printf("Save functionality is not implemented yet.\n");
            printf("Press any key to continue...");
            read(STDIN_FILENO, &c, 1);
        } else {
            write(STDOUT_FILENO, &c, 1);
        }
    }
}
```

**src/main.c**

```c
#include "editor.h"
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        return 1;
    }

    enable_raw_mode();
    
    printf("\x1b[H\x1b[J"); // 화면 지우기
    printf("Simple Text Editor -- %s\n", argv[1]);
    printf("Ctrl+S to save, Ctrl+Q to quit\n");

    editor_process_keypress();

    printf("\x1b[H\x1b[J"); // 화면 지우기
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/editor.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_text_editor

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`editor.h`)**:
    - 텍스트 에디터에 필요한 함수의 선언이 포함되어 있습니다.
    - `enable_raw_mode` 함수는 터미널을 RAW 모드로 설정합니다.
    - `disable_raw_mode` 함수는 터미널의 원래 모드로 복원합니다.
    - `editor_process_keypress` 함수는 키 입력을 처리합니다.

2. **소스 파일(`editor.c`)**:
    - `disable_raw_mode` 함수는 프로그램 종료 시 터미널 설정을 원래 상태로 복원합니다.
    - `enable_raw_mode` 함수는 터미널을 RAW 모드로 설정합니다.
        - RAW 모드에서는 터미널의 기본 입력 처리를 비활성화하여 키 입력을 직접 처리할 수 있습니다.
    - `editor_process_keypress` 함수는 사용자의 키 입력을 처리합니다.
        - `Ctrl+Q`를 누르면 프로그램을 종료합니다.
        - `Ctrl+S`를 누르면 저장 기능을 호출합니다(현재는 미구현 상태).
        - 기타 키 입력은 화면에 출력됩니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 파일 이름을 받습니다.
    - `enable_raw_mode` 함수를 호출하여 터미널을 RAW 모드로 설정합니다.
    - 간단한 안내 메시지를 출력한 후, `editor_process_keypress` 함수를 호출하여 키 입력을 처리합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

### 1.6 텍스트 에디터

#### 프로젝트 구조

```
text_editor/
├── include/
│   └── editor.h
├── src/
│   ├── editor.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 텍스트 에디터

이 프로젝트는 간단한 텍스트 에디터를 구현합니다. 파일을 열고, 내용을 편집하고, 저장할 수 있습니다.

## 파일 구조

- `include/editor.h` : 에디터 함수와 구조체 선언이 포함된 헤더 파일.
- `src/editor.c` : 에디터 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./text_editor <filename>
```

## 사용 예

프로그램을 실행하고, 파일의 내용을 편집합니다.

```sh
$ ./text_editor example.txt
```
```

#### 코드 구현 및 상세 설명

**include/editor.h**

```c
#ifndef EDITOR_H
#define EDITOR_H

#define MAX_LINE_LENGTH 1024

// 에디터 함수 선언
void open_file(const char *filename);
void save_file(const char *filename);
void edit_file();

#endif // EDITOR_H
```

**src/editor.c**

```c
#include "editor.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static char buffer[MAX_LINE_LENGTH];

void open_file(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        perror("Failed to open file");
        return;
    }

    while (fgets(buffer, sizeof(buffer), file)) {
        printf("%s", buffer);
    }

    fclose(file);
}

void save_file(const char *filename) {
    FILE *file = fopen(filename, "w");
    if (file == NULL) {
        perror("Failed to save file");
        return;
    }

    fputs(buffer, file);

    fclose(file);
}

void edit_file() {
    printf("Enter your text below (end with a single '.' on a new line):\n");
    int index = 0;
    while (1) {
        char line[MAX_LINE_LENGTH];
        fgets(line, sizeof(line), stdin);
        if (strcmp(line, ".\n") == 0) {
            break;
        }
        strncpy(buffer + index, line, MAX_LINE_LENGTH - index);
        index += strlen(line);
        if (index >= MAX_LINE_LENGTH) {
            printf("Buffer overflow, only partial content will be saved.\n");
            break;
        }
    }
}
```

**src/main.c**

```c
#include "editor.h"
#include <stdio.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        return 1;
    }

    const char *filename = argv[1];

    printf("Opening file: %s\n", filename);
    open_file(filename);

    printf("\nEditing file: %s\n", filename);
    edit_file();

    printf("\nSaving file: %s\n", filename);
    save_file(filename);

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/editor.c
OBJS = $(SRCS:.c=.o)

TARGET = text_editor

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`editor.h`)**:
    - 텍스트 에디터에 필요한 함수의 선언이 포함되어 있습니다.
    - `open_file`, `save_file`, `edit_file` 함수가 선언되어 있습니다.

2. **소스 파일(`editor.c`)**:
    - `open_file` 함수는 파일을 열고 내용을 출력합니다.
    - `save_file` 함수는 버퍼의 내용을 파일에 저장합니다.
    - `edit_file` 함수는 사용자가 입력한 텍스트를 버퍼에 저장합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 파일 이름을 받습니다.
    - `open_file` 함수를 호출하여 파일을 엽니다.
    - `edit_file` 함수를 호출하여 파일을 편집합니다.
    - `save_file` 함수를 호출하여 파일을 저장합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
