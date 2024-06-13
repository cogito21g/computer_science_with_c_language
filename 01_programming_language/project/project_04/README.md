# Project 04

### 1.4 데이터 파싱 프로그램

#### 프로젝트 구조

```
data_parser/
├── include/
│   └── parser.h
├── src/
│   ├── parser.c
│   └── main.c
├── data/
│   └── sample.csv
├── Makefile
└── README.md
```

#### README.md

```markdown
# 데이터 파싱 프로그램

이 프로젝트는 CSV 파일을 읽고 파싱하여 데이터를 처리하는 간단한 프로그램입니다. 프로그램은 각 행의 데이터를 파싱하여 출력합니다.

## 파일 구조

- `include/parser.h` : 파서 함수와 구조체 선언이 포함된 헤더 파일.
- `src/parser.c` : 파서 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `data/sample.csv` : 파싱할 샘플 CSV 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./data_parser data/sample.csv
```

## 사용 예

프로그램을 실행하고, CSV 파일의 내용을 파싱하여 출력합니다.

```sh
$ ./data_parser data/sample.csv
Name, Age, Occupation
Alice, 30, Engineer
Bob, 25, Designer
Charlie, 35, Teacher
```
```

#### 코드 구현 및 상세 설명

**include/parser.h**

```c
#ifndef PARSER_H
#define PARSER_H

#define MAX_LINE_LENGTH 1024
#define MAX_FIELDS 10

// CSV 행 구조체
typedef struct {
    char *fields[MAX_FIELDS];
    int num_fields;
} CSVRow;

// CSV 파일 파싱 함수
int parse_csv(const char *filename);

#endif // PARSER_H
```

**src/parser.c**

```c
#include "parser.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// CSV 행 파싱 함수
static CSVRow parse_line(char *line) {
    CSVRow row;
    row.num_fields = 0;
    char *token = strtok(line, ",");
    while (token != NULL && row.num_fields < MAX_FIELDS) {
        row.fields[row.num_fields++] = token;
        token = strtok(NULL, ",");
    }
    return row;
}

// CSV 파일 파싱 함수
int parse_csv(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        perror("Failed to open file");
        return -1;
    }

    char line[MAX_LINE_LENGTH];
    while (fgets(line, sizeof(line), file)) {
        // 개행 문자 제거
        line[strcspn(line, "\n")] = '\0';
        
        CSVRow row = parse_line(line);

        // 파싱된 행 출력
        for (int i = 0; i < row.num_fields; i++) {
            if (i > 0) {
                printf(", ");
            }
            printf("%s", row.fields[i]);
        }
        printf("\n");
    }

    fclose(file);
    return 0;
}
```

**src/main.c**

```c
#include "parser.h"
#include <stdio.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <csv_file>\n", argv[0]);
        return 1;
    }

    const char *filename = argv[1];
    if (parse_csv(filename) != 0) {
        fprintf(stderr, "Failed to parse CSV file: %s\n", filename);
        return 1;
    }

    return 0;
}
```

**data/sample.csv**

```
Name,Age,Occupation
Alice,30,Engineer
Bob,25,Designer
Charlie,35,Teacher
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/parser.c
OBJS = $(SRCS:.c=.o)

TARGET = data_parser

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`parser.h`)**:
    - 파서에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - `CSVRow` 구조체는 CSV 행의 데이터를 저장합니다.
    - `parse_csv` 함수는 CSV 파일을 파싱하는 함수입니다.

2. **소스 파일(`parser.c`)**:
    - `parse_line` 함수는 CSV 파일의 한 행을 파싱하여 `CSVRow` 구조체에 저장합니다.
    - `parse_csv` 함수는 CSV 파일을 읽고 각 행을 파싱하여 출력합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 명령줄 인수로 CSV 파일 이름을 받습니다.
    - `parse_csv` 함수를 호출하여 CSV 파일을 파싱합니다.

4. **샘플 CSV 파일(`sample.csv`)**:
    - 파싱할 샘플 CSV 파일입니다. 각 행에는 이름, 나이, 직업이 포함되어 있습니다.

5. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
