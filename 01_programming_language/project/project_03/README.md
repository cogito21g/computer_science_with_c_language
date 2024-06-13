# Project 03

### 1.3 간단한 파일 시스템

#### 프로젝트 구조

```
simple_file_system/
├── include/
│   └── filesystem.h
├── src/
│   ├── filesystem.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 간단한 파일 시스템

이 프로젝트는 메모리에 간단한 파일 시스템을 구현합니다. 파일을 생성, 삭제, 읽기, 쓰기 기능을 포함합니다.

## 파일 구조

- `include/filesystem.h` : 파일 시스템 함수와 구조체 선언이 포함된 헤더 파일.
- `src/filesystem.c` : 파일 시스템 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./simple_file_system
```

## 사용 예

프로그램을 실행하고, 파일 시스템 명령을 입력합니다.

```sh
> create myfile
File 'myfile' created.
> write myfile Hello, World!
> read myfile
Hello, World!
> delete myfile
File 'myfile' deleted.
> exit
```
```

#### 코드 구현 및 상세 설명

**include/filesystem.h**

```c
#ifndef FILESYSTEM_H
#define FILESYSTEM_H

#define MAX_FILES 100
#define MAX_FILENAME 100
#define MAX_FILESIZE 1024

typedef struct {
    char name[MAX_FILENAME];
    char data[MAX_FILESIZE];
    int size;
    int in_use;
} File;

typedef struct {
    File files[MAX_FILES];
} FileSystem;

void initialize_filesystem(FileSystem *fs);
int create_file(FileSystem *fs, const char *name);
int delete_file(FileSystem *fs, const char *name);
int write_file(FileSystem *fs, const char *name, const char *data);
int read_file(FileSystem *fs, const char *name, char *buffer);

#endif // FILESYSTEM_H
```

**src/filesystem.c**

```c
#include "filesystem.h"
#include <string.h>
#include <stdio.h>

// 파일 시스템 초기화
void initialize_filesystem(FileSystem *fs) {
    for (int i = 0; i < MAX_FILES; i++) {
        fs->files[i].in_use = 0;
    }
}

// 파일 생성
int create_file(FileSystem *fs, const char *name) {
    for (int i = 0; i < MAX_FILES; i++) {
        if (!fs->files[i].in_use) {
            strncpy(fs->files[i].name, name, MAX_FILENAME);
            fs->files[i].size = 0;
            fs->files[i].in_use = 1;
            return 0;
        }
    }
    return -1; // 파일 시스템 가득 참
}

// 파일 삭제
int delete_file(FileSystem *fs, const char *name) {
    for (int i = 0; i < MAX_FILES; i++) {
        if (fs->files[i].in_use && strcmp(fs->files[i].name, name) == 0) {
            fs->files[i].in_use = 0;
            return 0;
        }
    }
    return -1; // 파일을 찾을 수 없음
}

// 파일 쓰기
int write_file(FileSystem *fs, const char *name, const char *data) {
    for (int i = 0; i < MAX_FILES; i++) {
        if (fs->files[i].in_use && strcmp(fs->files[i].name, name) == 0) {
            int data_len = strlen(data);
            if (data_len > MAX_FILESIZE) {
                return -1; // 데이터 크기 초과
            }
            strncpy(fs->files[i].data, data, MAX_FILESIZE);
            fs->files[i].size = data_len;
            return 0;
        }
    }
    return -1; // 파일을 찾을 수 없음
}

// 파일 읽기
int read_file(FileSystem *fs, const char *name, char *buffer) {
    for (int i = 0; i < MAX_FILES; i++) {
        if (fs->files[i].in_use && strcmp(fs->files[i].name, name) == 0) {
            strncpy(buffer, fs->files[i].data, fs->files[i].size);
            buffer[fs->files[i].size] = '\0';
            return 0;
        }
    }
    return -1; // 파일을 찾을 수 없음
}
```

**src/main.c**

```c
#include "filesystem.h"
#include <stdio.h>
#include <string.h>

void print_help() {
    printf("Available commands:\n");
    printf("  create <filename> - Create a new file\n");
    printf("  delete <filename> - Delete a file\n");
    printf("  write <filename> <data> - Write data to a file\n");
    printf("  read <filename> - Read data from a file\n");
    printf("  exit - Exit the program\n");
}

int main() {
    FileSystem fs;
    char command[256];
    char name[MAX_FILENAME];
    char data[MAX_FILESIZE];
    char buffer[MAX_FILESIZE];

    initialize_filesystem(&fs);

    printf("Welcome to the simple file system!\n");
    print_help();

    while (1) {
        printf("> ");
        if (fgets(command, sizeof(command), stdin) != NULL) {
            command[strcspn(command, "\n")] = 0; // 개행 문자 제거

            if (strncmp(command, "create ", 7) == 0) {
                sscanf(command + 7, "%s", name);
                if (create_file(&fs, name) == 0) {
                    printf("File '%s' created.\n", name);
                } else {
                    printf("Failed to create file '%s'.\n", name);
                }
            } else if (strncmp(command, "delete ", 7) == 0) {
                sscanf(command + 7, "%s", name);
                if (delete_file(&fs, name) == 0) {
                    printf("File '%s' deleted.\n", name);
                } else {
                    printf("Failed to delete file '%s'.\n", name);
                }
            } else if (strncmp(command, "write ", 6) == 0) {
                sscanf(command + 6, "%s %[^\n]", name, data);
                if (write_file(&fs, name, data) == 0) {
                    printf("Data written to file '%s'.\n", name);
                } else {
                    printf("Failed to write to file '%s'.\n", name);
                }
            } else if (strncmp(command, "read ", 5) == 0) {
                sscanf(command + 5, "%s", name);
                if (read_file(&fs, name, buffer) == 0) {
                    printf("Data from file '%s': %s\n", name, buffer);
                } else {
                    printf("Failed to read from file '%s'.\n", name);
                }
            } else if (strcmp(command, "exit") == 0) {
                printf("Exiting the program.\n");
                break;
            } else {
                printf("Unknown command: %s\n", command);
                print_help();
            }
        }
    }

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/filesystem.c
OBJS = $(SRCS:.c=.o)

TARGET = simple_file_system

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`filesystem.h`)**:
    - 파일 시스템에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - `File` 구조체는 파일의 이름, 데이터, 크기, 사용 여부를 정의합니다.
    - `FileSystem` 구조체는 파일 시스템의 파일 목록을 정의합니다.
    - 각 함수의 선언이 포함되어 있습니다.

2. **소스 파일(`filesystem.c`)**:
    - `initialize_filesystem` 함수는 파일 시스템을 초기화합니다.
    - `create_file` 함수는 파일을 생성합니다.
    - `delete_file` 함수는 파일을 삭제합니다.
    - `write_file` 함수는 파일에 데이터를 씁니다.
    - `read_file` 함수는 파일에서 데이터를 읽습니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 사용자로부터 명령을 입력받아 처리합니다.
    - `print_help` 함수는 사용 가능한 명령어를 출력합니다.
    - `fgets`를 사용하여 사용자 입력을 받고, 명령어에 따라 파일 시스템 함수를 호출합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일

을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
