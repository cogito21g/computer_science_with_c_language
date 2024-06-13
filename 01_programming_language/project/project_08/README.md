# Project 08

### 1.8 과학 계산 프로그램

#### 프로젝트 구조

```
scientific_calculator/
├── include/
│   └── calculator.h
├── src/
│   ├── calculator.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 과학 계산 프로그램

이 프로젝트는 기본적인 과학 계산 기능을 제공하는 프로그램입니다. 삼각 함수, 로그 함수, 지수 함수 등을 포함합니다.

## 파일 구조

- `include/calculator.h` : 계산기 함수와 구조체 선언이 포함된 헤더 파일.
- `src/calculator.c` : 계산기 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./scientific_calculator
```

## 사용 예

프로그램을 실행하고, 과학 계산 명령어를 입력합니다.

```sh
$ ./scientific_calculator
Enter command: sin 0.5
Result: 0.479426
Enter command: cos 0.5
Result: 0.877583
Enter command: exp 1
Result: 2.718282
Enter command: log 10
Result: 2.302585
Enter command: exit
```
```

#### 코드 구현 및 상세 설명

**include/calculator.h**

```c
#ifndef CALCULATOR_H
#define CALCULATOR_H

#define MAX_COMMAND_LENGTH 1024

// 계산기 함수 선언
void start_calculator();
void execute_command(char *command);
void parse_command(char *command, char **args);
double calculate_sin(double x);
double calculate_cos(double x);
double calculate_exp(double x);
double calculate_log(double x);

#endif // CALCULATOR_H
```

**src/calculator.c**

```c
#include "calculator.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

void start_calculator() {
    char command[MAX_COMMAND_LENGTH];

    while (1) {
        printf("Enter command: ");
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
    parse_command(command, args);

    if (strcmp(args[0], "sin") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_sin(x));
        } else {
            printf("Error: Missing argument for sin\n");
        }
    } else if (strcmp(args[0], "cos") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_cos(x));
        } else {
            printf("Error: Missing argument for cos\n");
        }
    } else if (strcmp(args[0], "exp") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_exp(x));
        } else {
            printf("Error: Missing argument for exp\n");
        }
    } else if (strcmp(args[0], "log") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_log(x));
        } else {
            printf("Error: Missing argument for log\n");
        }
    } else {
        printf("Unknown command: %s\n", args[0]);
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

double calculate_sin(double x) {
    return sin(x);
}

double calculate_cos(double x) {
    return cos(x);
}

double calculate_exp(double x) {
    return exp(x);
}

double calculate_log(double x) {
    return log(x);
}
```

**src/main.c**

```c
#include "calculator.h"

int main() {
    start_calculator();
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall -lm

SRCS = src/main.c src/calculator.c
OBJS = $(SRCS:.c=.o)

TARGET = scientific_calculator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS) $(CFLAGS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`calculator.h`)**:
    - 과학 계산 프로그램에 필요한 함수와 상수의 선언이 포함되어 있습니다.
    - `MAX_COMMAND_LENGTH`는 명령어의 최대 길이를 정의합니다.
    - 계산기 함수들이 선언되어 있습니다.

2. **소스 파일(`calculator.c`)**:
    - `start_calculator` 함수는 계산기를 시작하고 사용자 입력을 처리합니다.
        - 무한 루프를 돌면서 사용자로부터 명령어를 입력받습니다.
        - 입력받은 명령어가 `exit`이면 루프를 종료합니다.
        - 입력받은 명령어를 `execute_command` 함수로 실행합니다.
    - `execute_command` 함수는 명령어를 실행합니다.
        - 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
        - 명령어에 따라 적절한 계산 함수를 호출하고 결과를 출력합니다.
    - `parse_command` 함수는 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
    - `calculate_sin`, `calculate_cos`, `calculate_exp`, `calculate_log` 함수는 각각 삼각 함수, 지수 함수, 로그 함수를 계산합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, `start_calculator` 함수를 호출하여 계산기를 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시키고 `-lm` 플래그를 사용하여 수학 라이브러리를 링크합니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

---

## 추가할 기능

1. 추가적인 수학 함수 지원: `tan`, `sqrt`, `pow`, `log10`, `abs`
2. 사용자 정의 함수 지원
3. 기본적인 변수 저장 및 참조 기능
4. 파일에서 명령어 읽기 기능

### 프로젝트 구조

```
scientific_calculator/
├── include/
│   └── calculator.h
├── src/
│   ├── calculator.c
│   └── main.c
├── Makefile
└── README.md
```

### README.md

```markdown
# 과학 계산 프로그램

이 프로젝트는 기본적인 과학 계산 기능을 제공하는 프로그램입니다. 삼각 함수, 로그 함수, 지수 함수 등을 포함합니다. 추가적으로 사용자 정의 함수, 변수 저장 및 참조, 파일에서 명령어 읽기 기능을 지원합니다.

## 파일 구조

- `include/calculator.h` : 계산기 함수와 구조체 선언이 포함된 헤더 파일.
- `src/calculator.c` : 계산기 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./scientific_calculator
```

## 사용 예

프로그램을 실행하고, 과학 계산 명령어를 입력합니다.

```sh
$ ./scientific_calculator
Enter command: sin 0.5
Result: 0.479426
Enter command: cos 0.5
Result: 0.877583
Enter command: exp 1
Result: 2.718282
Enter command: log 10
Result: 2.302585
Enter command: sqrt 16
Result: 4.000000
Enter command: pow 2 3
Result: 8.000000
Enter command: set x 10
Enter command: x
Result: 10.000000
Enter command: exit
```
```

### 코드 구현 및 상세 설명

**include/calculator.h**

```c
#ifndef CALCULATOR_H
#define CALCULATOR_H

#define MAX_COMMAND_LENGTH 1024
#define MAX_VARIABLES 100
#define MAX_NAME_LENGTH 50

typedef struct {
    char name[MAX_NAME_LENGTH];
    double value;
} Variable;

// 계산기 함수 선언
void start_calculator();
void execute_command(char *command);
void parse_command(char *command, char **args);
double calculate_sin(double x);
double calculate_cos(double x);
double calculate_tan(double x);
double calculate_exp(double x);
double calculate_log(double x);
double calculate_log10(double x);
double calculate_sqrt(double x);
double calculate_pow(double base, double exp);
double calculate_abs(double x);
void set_variable(char *name, double value);
double get_variable(char *name);
void list_variables();
void load_commands_from_file(const char *filename);

#endif // CALCULATOR_H
```

**src/calculator.c**

```c
#include "calculator.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

static Variable variables[MAX_VARIABLES];
static int variable_count = 0;

void start_calculator() {
    char command[MAX_COMMAND_LENGTH];

    while (1) {
        printf("Enter command: ");
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
    parse_command(command, args);

    if (strcmp(args[0], "sin") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_sin(x));
        } else {
            printf("Error: Missing argument for sin\n");
        }
    } else if (strcmp(args[0], "cos") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_cos(x));
        } else {
            printf("Error: Missing argument for cos\n");
        }
    } else if (strcmp(args[0], "tan") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_tan(x));
        } else {
            printf("Error: Missing argument for tan\n");
        }
    } else if (strcmp(args[0], "exp") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_exp(x));
        } else {
            printf("Error: Missing argument for exp\n");
        }
    } else if (strcmp(args[0], "log") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_log(x));
        } else {
            printf("Error: Missing argument for log\n");
        }
    } else if (strcmp(args[0], "log10") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_log10(x));
        } else {
            printf("Error: Missing argument for log10\n");
        }
    } else if (strcmp(args[0], "sqrt") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_sqrt(x));
        } else {
            printf("Error: Missing argument for sqrt\n");
        }
    } else if (strcmp(args[0], "pow") == 0) {
        if (args[1] != NULL && args[2] != NULL) {
            double base = atof(args[1]);
            double exp = atof(args[2]);
            printf("Result: %f\n", calculate_pow(base, exp));
        } else {
            printf("Error: Missing argument for pow\n");
        }
    } else if (strcmp(args[0], "abs") == 0) {
        if (args[1] != NULL) {
            double x = atof(args[1]);
            printf("Result: %f\n", calculate_abs(x));
        } else {
            printf("Error: Missing argument for abs\n");
        }
    } else if (strcmp(args[0], "set") == 0) {
        if (args[1] != NULL && args[2] != NULL) {
            char *name = args[1];
            double value = atof(args[2]);
            set_variable(name, value);
            printf("Variable '%s' set to %f\n", name, value);
        } else {
            printf("Error: Missing argument for set\n");
        }
    } else if (strcmp(args[0], "list") == 0) {
        list_variables();
    } else if (args[0][0] == '@') {
        load_commands_from_file(args[0] + 1);
    } else {
        double value = get_variable(args[0]);
        if (!isnan(value)) {
            printf("Result: %f\n", value);
        } else {
            printf("Unknown command or variable: %s\n", args[0]);
        }
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

double calculate_sin(double x) {
    return sin(x);
}

double calculate_cos(double x) {
    return cos(x);
}

double calculate_tan(double x) {
    return tan(x);
}

double calculate_exp(double x) {
    return exp(x);
}

double calculate_log(double x) {
    return log(x);
}

double calculate_log10(double x) {
    return log10(x);
}

double calculate_sqrt(double x) {
    return sqrt(x);
}

double calculate_pow(double base, double exp) {
    return pow(base, exp);
}

double calculate_abs(double x) {
    return fabs(x);
}

void set_variable(char *name, double value) {
    for (int i = 0; i < variable_count; i++) {
        if (strcmp(variables[i].name, name) == 0) {
            variables[i].value = value;
            return;
        }
    }
    if (variable_count < MAX_VARIABLES) {
        strncpy(variables[variable_count].name, name, MAX_NAME_LENGTH);
        variables[variable_count].value = value;
        variable_count++;
    } else {
        printf("Error: Variable storage limit reached\n");
    }
}

double get_variable(char *name) {
    for (int i = 0; i < variable

_count; i++) {
        if (strcmp(variables[i].name, name) == 0) {
            return variables[i].value;
        }
    }
    return NAN;
}

void list_variables() {
    for (int i = 0; i < variable_count; i++) {
        printf("%s = %f\n", variables[i].name, variables[i].value);
    }
}

void load_commands_from_file(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        perror("Failed to open file");
        return;
    }

    char line[MAX_COMMAND_LENGTH];
    while (fgets(line, sizeof(line), file)) {
        // 개행 문자 제거
        line[strcspn(line, "\n")] = '\0';

        // 명령어 실행
        execute_command(line);
    }

    fclose(file);
}
```

**src/main.c**

```c
#include "calculator.h"

int main() {
    start_calculator();
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall -lm

SRCS = src/main.c src/calculator.c
OBJS = $(SRCS:.c=.o)

TARGET = scientific_calculator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS) $(CFLAGS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`calculator.h`)**:
    - 과학 계산 프로그램에 필요한 함수와 상수의 선언이 포함되어 있습니다.
    - `MAX_COMMAND_LENGTH`는 명령어의 최대 길이를 정의합니다.
    - `MAX_VARIABLES`는 저장할 수 있는 변수의 최대 개수를 정의합니다.
    - `Variable` 구조체는 변수의 이름과 값을 저장합니다.
    - 계산기 함수들이 선언되어 있습니다.

2. **소스 파일(`calculator.c`)**:
    - `start_calculator` 함수는 계산기를 시작하고 사용자 입력을 처리합니다.
        - 무한 루프를 돌면서 사용자로부터 명령어를 입력받습니다.
        - 입력받은 명령어가 `exit`이면 루프를 종료합니다.
        - 입력받은 명령어를 `execute_command` 함수로 실행합니다.
    - `execute_command` 함수는 명령어를 실행합니다.
        - 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
        - 명령어에 따라 적절한 계산 함수를 호출하고 결과를 출력합니다.
        - 변수 설정, 변수 출력, 파일로부터 명령어 읽기 등의 기능을 처리합니다.
    - `parse_command` 함수는 명령어를 공백으로 분할하여 `args` 배열에 저장합니다.
    - `calculate_sin`, `calculate_cos`, `calculate_tan`, `calculate_exp`, `calculate_log`, `calculate_log10`, `calculate_sqrt`, `calculate_pow`, `calculate_abs` 함수는 각각의 수학 함수를 계산합니다.
    - `set_variable` 함수는 변수를 설정합니다.
    - `get_variable` 함수는 변수를 조회합니다.
    - `list_variables` 함수는 모든 변수를 출력합니다.
    - `load_commands_from_file` 함수는 파일에서 명령어를 읽어 실행합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, `start_calculator` 함수를 호출하여 계산기를 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시키고 `-lm` 플래그를 사용하여 수학 라이브러리를 링크합니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
