# Project 01: Calculator

### 1.1 계산기 프로그램

#### 프로젝트 구조

```
calculator/
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
# 계산기 프로그램

이 프로젝트는 기본적인 사칙연산(덧셈, 뺄셈, 곱셈, 나눗셈)을 수행하는 간단한 계산기 프로그램입니다.

## 파일 구조

- `include/calculator.h` : 계산기 함수의 선언이 포함된 헤더 파일.
- `src/calculator.c` : 계산기 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./calculator
```

## 사용 예

프로그램을 실행하고, 연산을 수행할 두 숫자와 연산자를 입력합니다.

```sh
Enter first number: 10
Enter an operator (+, -, *, /): *
Enter second number: 5
Result: 50
```
```

#### 코드 구현 및 상세 설명

**include/calculator.h**

```c
#ifndef CALCULATOR_H
#define CALCULATOR_H

// 사칙연산 함수 선언
double add(double a, double b);
double subtract(double a, double b);
double multiply(double a, double b);
double divide(double a, double b);

#endif // CALCULATOR_H
```

**src/calculator.c**

```c
#include "calculator.h"

// 덧셈 함수
double add(double a, double b) {
    return a + b;
}

// 뺄셈 함수
double subtract(double a, double b) {
    return a - b;
}

// 곱셈 함수
double multiply(double a, double b) {
    return a * b;
}

// 나눗셈 함수 (0으로 나누기 처리)
double divide(double a, double b) {
    if (b == 0) {
        return 0; // 실제 응용 프로그램에서는 오류를 반환하거나 예외를 발생시켜야 합니다.
    }
    return a / b;
}
```

**src/main.c**

```c
#include <stdio.h>
#include "calculator.h"

int main() {
    double num1, num2, result;
    char operator;

    // 사용자 입력 받기
    printf("Enter first number: ");
    scanf("%lf", &num1);

    printf("Enter an operator (+, -, *, /): ");
    scanf(" %c", &operator);

    printf("Enter second number: ");
    scanf("%lf", &num2);

    // 연산자에 따른 계산 수행
    switch (operator) {
        case '+':
            result = add(num1, num2);
            break;
        case '-':
            result = subtract(num1, num2);
            break;
        case '*':
            result = multiply(num1, num2);
            break;
        case '/':
            result = divide(num1, num2);
            if (num2 == 0) {
                printf("Error: Division by zero.\n");
                return 1;
            }
            break;
        default:
            printf("Error: Invalid operator.\n");
            return 1;
    }

    // 결과 출력
    printf("Result: %lf\n", result);

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/calculator.c
OBJS = $(SRCS:.c=.o)

TARGET = calculator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`calculator.h`)**:
    - 계산기 함수들의 선언이 포함되어 있습니다.

2. **소스 파일(`calculator.c`)**:
    - 각 사칙연산 함수의 구현이 포함되어 있습니다.
    - `add`, `subtract`, `multiply`, `divide` 함수가 정의되어 있습니다.
    - `divide` 함수에서는 0으로 나누는 경우를 처리합니다.

3. **메인 파일(`main.c`)**:
    - 사용자로부터 입력을 받아 적절한 사칙연산 함수를 호출하여 결과를 출력합니다.
    - `scanf` 함수를 사용하여 사용자 입력을 처리합니다.
    - `switch` 문을 사용하여 연산자를 검사하고, 해당하는 연산 함수를 호출합니다.
    - 나눗셈의 경우 0으로 나누기를 검사하여 오류 메시지를 출력합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
