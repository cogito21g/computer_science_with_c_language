# Project 09

### 1.9 데이터 구조 시뮬레이터

#### 프로젝트 구조

```
data_structure_simulator/
├── include/
│   ├── stack.h
│   ├── queue.h
│   └── linked_list.h
├── src/
│   ├── stack.c
│   ├── queue.c
│   ├── linked_list.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 데이터 구조 시뮬레이터

이 프로젝트는 스택, 큐, 링크드 리스트 등의 기본 데이터 구조를 시뮬레이션하는 프로그램입니다. 사용자는 데이터 구조를 생성하고, 삽입, 삭제 등의 작업을 수행할 수 있습니다.

## 파일 구조

- `include/stack.h` : 스택 함수와 구조체 선언이 포함된 헤더 파일.
- `include/queue.h` : 큐 함수와 구조체 선언이 포함된 헤더 파일.
- `include/linked_list.h` : 링크드 리스트 함수와 구조체 선언이 포함된 헤더 파일.
- `src/stack.c` : 스택 함수의 구현이 포함된 소스 파일.
- `src/queue.c` : 큐 함수의 구현이 포함된 소스 파일.
- `src/linked_list.c` : 링크드 리스트 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./data_structure_simulator
```

## 사용 예

프로그램을 실행하고, 데이터 구조 명령어를 입력합니다.

```sh
$ ./data_structure_simulator
Enter command: create stack
Stack created.
Enter command: push 10
Pushed 10 to stack.
Enter command: push 20
Pushed 20 to stack.
Enter command: pop
Popped 20 from stack.
Enter command: display stack
Stack: 10
Enter command: exit
```
```

#### 코드 구현 및 상세 설명

**include/stack.h**

```c
#ifndef STACK_H
#define STACK_H

#define MAX_STACK_SIZE 100

typedef struct {
    int data[MAX_STACK_SIZE];
    int top;
} Stack;

// 스택 함수 선언
void init_stack(Stack *stack);
int is_stack_empty(Stack *stack);
int is_stack_full(Stack *stack);
void push(Stack *stack, int value);
int pop(Stack *stack);
void display_stack(Stack *stack);

#endif // STACK_H
```

**include/queue.h**

```c
#ifndef QUEUE_H
#define QUEUE_H

#define MAX_QUEUE_SIZE 100

typedef struct {
    int data[MAX_QUEUE_SIZE];
    int front;
    int rear;
} Queue;

// 큐 함수 선언
void init_queue(Queue *queue);
int is_queue_empty(Queue *queue);
int is_queue_full(Queue *queue);
void enqueue(Queue *queue, int value);
int dequeue(Queue *queue);
void display_queue(Queue *queue);

#endif // QUEUE_H
```

**include/linked_list.h**

```c
#ifndef LINKED_LIST_H
#define LINKED_LIST_H

typedef struct Node {
    int data;
    struct Node *next;
} Node;

// 링크드 리스트 함수 선언
Node* create_node(int data);
void insert_front(Node **head, int data);
void insert_end(Node **head, int data);
void delete_node(Node **head, int data);
void display_list(Node *head);

#endif // LINKED_LIST_H
```

**src/stack.c**

```c
#include "stack.h"
#include <stdio.h>

void init_stack(Stack *stack) {
    stack->top = -1;
}

int is_stack_empty(Stack *stack) {
    return stack->top == -1;
}

int is_stack_full(Stack *stack) {
    return stack->top == MAX_STACK_SIZE - 1;
}

void push(Stack *stack, int value) {
    if (is_stack_full(stack)) {
        printf("Stack overflow\n");
        return;
    }
    stack->data[++stack->top] = value;
}

int pop(Stack *stack) {
    if (is_stack_empty(stack)) {
        printf("Stack underflow\n");
        return -1;
    }
    return stack->data[stack->top--];
}

void display_stack(Stack *stack) {
    if (is_stack_empty(stack)) {
        printf("Stack is empty\n");
        return;
    }
    printf("Stack: ");
    for (int i = 0; i <= stack->top; i++) {
        printf("%d ", stack->data[i]);
    }
    printf("\n");
}
```

**src/queue.c**

```c
#include "queue.h"
#include <stdio.h>

void init_queue(Queue *queue) {
    queue->front = 0;
    queue->rear = -1;
}

int is_queue_empty(Queue *queue) {
    return queue->front > queue->rear;
}

int is_queue_full(Queue *queue) {
    return queue->rear == MAX_QUEUE_SIZE - 1;
}

void enqueue(Queue *queue, int value) {
    if (is_queue_full(queue)) {
        printf("Queue overflow\n");
        return;
    }
    queue->data[++queue->rear] = value;
}

int dequeue(Queue *queue) {
    if (is_queue_empty(queue)) {
        printf("Queue underflow\n");
        return -1;
    }
    return queue->data[queue->front++];
}

void display_queue(Queue *queue) {
    if (is_queue_empty(queue)) {
        printf("Queue is empty\n");
        return;
    }
    printf("Queue: ");
    for (int i = queue->front; i <= queue->rear; i++) {
        printf("%d ", queue->data[i]);
    }
    printf("\n");
}
```

**src/linked_list.c**

```c
#include "linked_list.h"
#include <stdio.h>
#include <stdlib.h>

Node* create_node(int data) {
    Node *new_node = (Node *)malloc(sizeof(Node));
    if (!new_node) {
        printf("Memory allocation failed\n");
        exit(1);
    }
    new_node->data = data;
    new_node->next = NULL;
    return new_node;
}

void insert_front(Node **head, int data) {
    Node *new_node = create_node(data);
    new_node->next = *head;
    *head = new_node;
}

void insert_end(Node **head, int data) {
    Node *new_node = create_node(data);
    if (*head == NULL) {
        *head = new_node;
        return;
    }
    Node *temp = *head;
    while (temp->next) {
        temp = temp->next;
    }
    temp->next = new_node;
}

void delete_node(Node **head, int data) {
    if (*head == NULL) {
        printf("List is empty\n");
        return;
    }
    Node *temp = *head;
    if (temp->data == data) {
        *head = temp->next;
        free(temp);
        return;
    }
    Node *prev = NULL;
    while (temp && temp->data != data) {
        prev = temp;
        temp = temp->next;
    }
    if (temp == NULL) {
        printf("Data not found in the list\n");
        return;
    }
    prev->next = temp->next;
    free(temp);
}

void display_list(Node *head) {
    if (head == NULL) {
        printf("List is empty\n");
        return;
    }
    Node *temp = head;
    printf("Linked List: ");
    while (temp) {
        printf("%d ", temp->data);
        temp = temp->next;
    }
    printf("\n");
}
```

**src/main.c**

```c
#include "stack.h"
#include "queue.h"
#include "linked_list.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

Stack stack;
Queue queue;
Node *list = NULL;

void process_command(char *command) {
    char cmd[50];
    int value;
    sscanf(command, "%s %d", cmd, &value);

    if (strcmp(cmd, "create") == 0) {
        if (strcmp(command, "create stack") == 0) {
            init_stack(&stack);
            printf("Stack created.\n");
        } else if (strcmp(command, "create queue") == 0) {
            init_queue(&queue);
            printf("Queue created.\n");
        } else if (strcmp(command, "create list") == 0) {
            list = NULL;
            printf("Linked list created.\n");
        }
    } else if (strcmp(cmd, "push") == 0) {
        push(&stack, value);
        printf("Pushed %d to stack.\n", value);
    } else if (strcmp(cmd, "pop") == 0) {
        int popped = pop(&stack);
        if (popped != -1) {
            printf("Popped %d from stack.\n", popped);
        }
    } else if (strcmp(cmd, "enqueue") == 0) {
        enqueue(&queue, value);
        printf("

Enqueued %d to queue.\n", value);
    } else if (strcmp(cmd, "dequeue") == 0) {
        int dequeued = dequeue(&queue);
        if (dequeued != -1) {
            printf("Dequeued %d from queue.\n", dequeued);
        }
    } else if (strcmp(cmd, "insert_front") == 0) {
        insert_front(&list, value);
        printf("Inserted %d at front of list.\n", value);
    } else if (strcmp(cmd, "insert_end") == 0) {
        insert_end(&list, value);
        printf("Inserted %d at end of list.\n", value);
    } else if (strcmp(cmd, "delete") == 0) {
        delete_node(&list, value);
        printf("Deleted %d from list.\n", value);
    } else if (strcmp(cmd, "display") == 0) {
        if (strcmp(command, "display stack") == 0) {
            display_stack(&stack);
        } else if (strcmp(command, "display queue") == 0) {
            display_queue(&queue);
        } else if (strcmp(command, "display list") == 0) {
            display_list(list);
        }
    } else {
        printf("Unknown command.\n");
    }
}

int main() {
    char command[100];
    printf("Data Structure Simulator\n");
    printf("Enter command: ");
    
    while (fgets(command, sizeof(command), stdin)) {
        // 개행 문자 제거
        command[strcspn(command, "\n")] = '\0';
        process_command(command);
        printf("Enter command: ");
    }
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/stack.c src/queue.c src/linked_list.c
OBJS = $(SRCS:.c=.o)

TARGET = data_structure_simulator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일**:
    - `stack.h`, `queue.h`, `linked_list.h`는 각각 스택, 큐, 링크드 리스트에 필요한 구조체와 함수의 선언이 포함되어 있습니다.

2. **소스 파일**:
    - `stack.c`, `queue.c`, `linked_list.c`는 각각 스택, 큐, 링크드 리스트의 함수 구현이 포함되어 있습니다.
    - 각 파일은 초기화, 삽입, 삭제, 디스플레이 등의 기본 연산을 수행하는 함수들이 구현되어 있습니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 사용자의 명령어를 처리합니다.
    - `process_command` 함수는 사용자가 입력한 명령어를 분석하고, 적절한 데이터 구조 연산을 수행합니다.
    - 스택, 큐, 링크드 리스트를 생성하고, 삽입, 삭제, 디스플레이 등의 연산을 수행할 수 있습니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

--- 

### 추가 기능

1. **스택**:
    - Peek: 스택의 최상위 요소를 반환합니다.
    - Clear: 스택을 초기화합니다.

2. **큐**:
    - Peek: 큐의 첫 번째 요소를 반환합니다.
    - Clear: 큐를 초기화합니다.

3. **링크드 리스트**:
    - Search: 리스트에서 특정 값을 검색합니다.
    - Clear: 리스트를 초기화합니다.

#### 프로젝트 구조

```
data_structure_simulator/
├── include/
│   ├── stack.h
│   ├── queue.h
│   └── linked_list.h
├── src/
│   ├── stack.c
│   ├── queue.c
│   ├── linked_list.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 데이터 구조 시뮬레이터

이 프로젝트는 스택, 큐, 링크드 리스트 등의 기본 데이터 구조를 시뮬레이션하는 프로그램입니다. 사용자는 데이터 구조를 생성하고, 삽입, 삭제 등의 작업을 수행할 수 있습니다.

## 파일 구조

- `include/stack.h` : 스택 함수와 구조체 선언이 포함된 헤더 파일.
- `include/queue.h` : 큐 함수와 구조체 선언이 포함된 헤더 파일.
- `include/linked_list.h` : 링크드 리스트 함수와 구조체 선언이 포함된 헤더 파일.
- `src/stack.c` : 스택 함수의 구현이 포함된 소스 파일.
- `src/queue.c` : 큐 함수의 구현이 포함된 소스 파일.
- `src/linked_list.c` : 링크드 리스트 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./data_structure_simulator
```

## 사용 예

프로그램을 실행하고, 데이터 구조 명령어를 입력합니다.

```sh
$ ./data_structure_simulator
Enter command: create stack
Stack created.
Enter command: push 10
Pushed 10 to stack.
Enter command: peek stack
Top element is 10.
Enter command: clear stack
Stack cleared.
Enter command: display stack
Stack is empty.
Enter command: exit
```
```

### 코드 구현 및 상세 설명

**include/stack.h**

```c
#ifndef STACK_H
#define STACK_H

#define MAX_STACK_SIZE 100

typedef struct {
    int data[MAX_STACK_SIZE];
    int top;
} Stack;

// 스택 함수 선언
void init_stack(Stack *stack);
int is_stack_empty(Stack *stack);
int is_stack_full(Stack *stack);
void push(Stack *stack, int value);
int pop(Stack *stack);
int peek_stack(Stack *stack);
void clear_stack(Stack *stack);
void display_stack(Stack *stack);

#endif // STACK_H
```

**include/queue.h**

```c
#ifndef QUEUE_H
#define QUEUE_H

#define MAX_QUEUE_SIZE 100

typedef struct {
    int data[MAX_QUEUE_SIZE];
    int front;
    int rear;
} Queue;

// 큐 함수 선언
void init_queue(Queue *queue);
int is_queue_empty(Queue *queue);
int is_queue_full(Queue *queue);
void enqueue(Queue *queue, int value);
int dequeue(Queue *queue);
int peek_queue(Queue *queue);
void clear_queue(Queue *queue);
void display_queue(Queue *queue);

#endif // QUEUE_H
```

**include/linked_list.h**

```c
#ifndef LINKED_LIST_H
#define LINKED_LIST_H

typedef struct Node {
    int data;
    struct Node *next;
} Node;

// 링크드 리스트 함수 선언
Node* create_node(int data);
void insert_front(Node **head, int data);
void insert_end(Node **head, int data);
void delete_node(Node **head, int data);
Node* search_list(Node *head, int data);
void clear_list(Node **head);
void display_list(Node *head);

#endif // LINKED_LIST_H
```

**src/stack.c**

```c
#include "stack.h"
#include <stdio.h>

void init_stack(Stack *stack) {
    stack->top = -1;
}

int is_stack_empty(Stack *stack) {
    return stack->top == -1;
}

int is_stack_full(Stack *stack) {
    return stack->top == MAX_STACK_SIZE - 1;
}

void push(Stack *stack, int value) {
    if (is_stack_full(stack)) {
        printf("Stack overflow\n");
        return;
    }
    stack->data[++stack->top] = value;
}

int pop(Stack *stack) {
    if (is_stack_empty(stack)) {
        printf("Stack underflow\n");
        return -1;
    }
    return stack->data[stack->top--];
}

int peek_stack(Stack *stack) {
    if (is_stack_empty(stack)) {
        printf("Stack is empty\n");
        return -1;
    }
    return stack->data[stack->top];
}

void clear_stack(Stack *stack) {
    stack->top = -1;
}

void display_stack(Stack *stack) {
    if (is_stack_empty(stack)) {
        printf("Stack is empty\n");
        return;
    }
    printf("Stack: ");
    for (int i = 0; i <= stack->top; i++) {
        printf("%d ", stack->data[i]);
    }
    printf("\n");
}
```

**src/queue.c**

```c
#include "queue.h"
#include <stdio.h>

void init_queue(Queue *queue) {
    queue->front = 0;
    queue->rear = -1;
}

int is_queue_empty(Queue *queue) {
    return queue->front > queue->rear;
}

int is_queue_full(Queue *queue) {
    return queue->rear == MAX_QUEUE_SIZE - 1;
}

void enqueue(Queue *queue, int value) {
    if (is_queue_full(queue)) {
        printf("Queue overflow\n");
        return;
    }
    queue->data[++queue->rear] = value;
}

int dequeue(Queue *queue) {
    if (is_queue_empty(queue)) {
        printf("Queue underflow\n");
        return -1;
    }
    return queue->data[queue->front++];
}

int peek_queue(Queue *queue) {
    if (is_queue_empty(queue)) {
        printf("Queue is empty\n");
        return -1;
    }
    return queue->data[queue->front];
}

void clear_queue(Queue *queue) {
    queue->front = 0;
    queue->rear = -1;
}

void display_queue(Queue *queue) {
    if (is_queue_empty(queue)) {
        printf("Queue is empty\n");
        return;
    }
    printf("Queue: ");
    for (int i = queue->front; i <= queue->rear; i++) {
        printf("%d ", queue->data[i]);
    }
    printf("\n");
}
```

**src/linked_list.c**

```c
#include "linked_list.h"
#include <stdio.h>
#include <stdlib.h>

Node* create_node(int data) {
    Node *new_node = (Node *)malloc(sizeof(Node));
    if (!new_node) {
        printf("Memory allocation failed\n");
        exit(1);
    }
    new_node->data = data;
    new_node->next = NULL;
    return new_node;
}

void insert_front(Node **head, int data) {
    Node *new_node = create_node(data);
    new_node->next = *head;
    *head = new_node;
}

void insert_end(Node **head, int data) {
    Node *new_node = create_node(data);
    if (*head == NULL) {
        *head = new_node;
        return;
    }
    Node *temp = *head;
    while (temp->next) {
        temp = temp->next;
    }
    temp->next = new_node;
}

void delete_node(Node **head, int data) {
    if (*head == NULL) {
        printf("List is empty\n");
        return;
    }
    Node *temp = *head;
    if (temp->data == data) {
        *head = temp->next;
        free(temp);
        return;
    }
    Node *prev = NULL;
    while (temp && temp->data != data) {
        prev = temp;
        temp = temp->next;
    }
    if (temp == NULL) {
        printf("Data not found in the list\n");
        return;
    }
    prev->next = temp->next;
    free(temp);
}

Node* search_list(Node *head, int data) {
    Node *temp = head;
    while (temp) {
        if (temp->data == data) {
            return temp;
        }
        temp = temp->next;
    }
    return NULL;
}

void clear_list(Node **head) {
    Node *temp = *head;
    while (temp) {
        Node *next = temp->next;
        free(temp);
        temp = next;
    }
    *head = NULL;
}

void display_list(Node *head) {
    if (head == NULL) {
        printf("List is empty\n");
       

 return;
    }
    Node *temp = head;
    printf("Linked List: ");
    while (temp) {
        printf("%d ", temp->data);
        temp = temp->next;
    }
    printf("\n");
}
```

**src/main.c**

```c
#include "stack.h"
#include "queue.h"
#include "linked_list.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

Stack stack;
Queue queue;
Node *list = NULL;

void process_command(char *command) {
    char cmd[50];
    int value;
    sscanf(command, "%s %d", cmd, &value);

    if (strcmp(cmd, "create") == 0) {
        if (strcmp(command, "create stack") == 0) {
            init_stack(&stack);
            printf("Stack created.\n");
        } else if (strcmp(command, "create queue") == 0) {
            init_queue(&queue);
            printf("Queue created.\n");
        } else if (strcmp(command, "create list") == 0) {
            list = NULL;
            printf("Linked list created.\n");
        }
    } else if (strcmp(cmd, "push") == 0) {
        push(&stack, value);
        printf("Pushed %d to stack.\n", value);
    } else if (strcmp(cmd, "pop") == 0) {
        int popped = pop(&stack);
        if (popped != -1) {
            printf("Popped %d from stack.\n", popped);
        }
    } else if (strcmp(cmd, "peek") == 0) {
        if (strcmp(command, "peek stack") == 0) {
            int top = peek_stack(&stack);
            if (top != -1) {
                printf("Top element is %d.\n", top);
            }
        } else if (strcmp(command, "peek queue") == 0) {
            int front = peek_queue(&queue);
            if (front != -1) {
                printf("Front element is %d.\n", front);
            }
        }
    } else if (strcmp(cmd, "clear") == 0) {
        if (strcmp(command, "clear stack") == 0) {
            clear_stack(&stack);
            printf("Stack cleared.\n");
        } else if (strcmp(command, "clear queue") == 0) {
            clear_queue(&queue);
            printf("Queue cleared.\n");
        } else if (strcmp(command, "clear list") == 0) {
            clear_list(&list);
            printf("List cleared.\n");
        }
    } else if (strcmp(cmd, "enqueue") == 0) {
        enqueue(&queue, value);
        printf("Enqueued %d to queue.\n", value);
    } else if (strcmp(cmd, "dequeue") == 0) {
        int dequeued = dequeue(&queue);
        if (dequeued != -1) {
            printf("Dequeued %d from queue.\n", dequeued);
        }
    } else if (strcmp(cmd, "insert_front") == 0) {
        insert_front(&list, value);
        printf("Inserted %d at front of list.\n", value);
    } else if (strcmp(cmd, "insert_end") == 0) {
        insert_end(&list, value);
        printf("Inserted %d at end of list.\n", value);
    } else if (strcmp(cmd, "delete") == 0) {
        delete_node(&list, value);
        printf("Deleted %d from list.\n", value);
    } else if (strcmp(cmd, "search") == 0) {
        Node *found = search_list(list, value);
        if (found) {
            printf("Found %d in list.\n", value);
        } else {
            printf("Did not find %d in list.\n", value);
        }
    } else if (strcmp(cmd, "display") == 0) {
        if (strcmp(command, "display stack") == 0) {
            display_stack(&stack);
        } else if (strcmp(command, "display queue") == 0) {
            display_queue(&queue);
        } else if (strcmp(command, "display list") == 0) {
            display_list(list);
        }
    } else {
        printf("Unknown command.\n");
    }
}

int main() {
    char command[100];
    printf("Data Structure Simulator\n");
    printf("Enter command: ");
    
    while (fgets(command, sizeof(command), stdin)) {
        // 개행 문자 제거
        command[strcspn(command, "\n")] = '\0';
        process_command(command);
        printf("Enter command: ");
    }
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/stack.c src/queue.c src/linked_list.c
OBJS = $(SRCS:.c=.o)

TARGET = data_structure_simulator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일**:
    - `stack.h`, `queue.h`, `linked_list.h`는 각각 스택, 큐, 링크드 리스트에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - 스택과 큐의 `peek` 및 `clear` 함수가 추가되었습니다.
    - 링크드 리스트의 `search` 및 `clear` 함수가 추가되었습니다.

2. **소스 파일**:
    - `stack.c`, `queue.c`, `linked_list.c`는 각각 스택, 큐, 링크드 리스트의 함수 구현이 포함되어 있습니다.
    - 각 파일에 새로운 함수들이 추가되었습니다:
        - 스택: `peek_stack`, `clear_stack`
        - 큐: `peek_queue`, `clear_queue`
        - 링크드 리스트: `search_list`, `clear_list`
    - 각 파일은 초기화, 삽입, 삭제, 디스플레이 등의 기본 연산을 수행하는 함수들이 구현되어 있습니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 사용자의 명령어를 처리합니다.
    - `process_command` 함수는 사용자가 입력한 명령어를 분석하고, 적절한 데이터 구조 연산을 수행합니다.
    - 스택, 큐, 링크드 리스트를 생성하고, 삽입, 삭제, 디스플레이 등의 연산을 수행할 수 있습니다.
    - 명령어에 따라 적절한 연산을 수행합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.


---

## deque 추가

데이터 구조 시뮬레이터에 덱(deque, double-ended queue)을 추가하여 구현. 덱은 양쪽 끝에서 요소를 추가하거나 제거할 수 있는 자료 구조.

### 프로젝트 구조

```
data_structure_simulator/
├── include/
│   ├── stack.h
│   ├── queue.h
│   ├── linked_list.h
│   └── deque.h
├── src/
│   ├── stack.c
│   ├── queue.c
│   ├── linked_list.c
│   ├── deque.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 데이터 구조 시뮬레이터

이 프로젝트는 스택, 큐, 링크드 리스트, 덱 등의 기본 데이터 구조를 시뮬레이션하는 프로그램입니다. 사용자는 데이터 구조를 생성하고, 삽입, 삭제 등의 작업을 수행할 수 있습니다.

## 파일 구조

- `include/stack.h` : 스택 함수와 구조체 선언이 포함된 헤더 파일.
- `include/queue.h` : 큐 함수와 구조체 선언이 포함된 헤더 파일.
- `include/linked_list.h` : 링크드 리스트 함수와 구조체 선언이 포함된 헤더 파일.
- `include/deque.h` : 덱 함수와 구조체 선언이 포함된 헤더 파일.
- `src/stack.c` : 스택 함수의 구현이 포함된 소스 파일.
- `src/queue.c` : 큐 함수의 구현이 포함된 소스 파일.
- `src/linked_list.c` : 링크드 리스트 함수의 구현이 포함된 소스 파일.
- `src/deque.c` : 덱 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./data_structure_simulator
```

## 사용 예

프로그램을 실행하고, 데이터 구조 명령어를 입력합니다.

```sh
$ ./data_structure_simulator
Enter command: create deque
Deque created.
Enter command: push_front 10
Pushed 10 to front of deque.
Enter command: push_back 20
Pushed 20 to back of deque.
Enter command: pop_front
Popped 10 from front of deque.
Enter command: pop_back
Popped 20 from back of deque.
Enter command: display deque
Deque is empty.
Enter command: exit
```
```

### 코드 구현 및 상세 설명

**include/deque.h**

```c
#ifndef DEQUE_H
#define DEQUE_H

#define MAX_DEQUE_SIZE 100

typedef struct {
    int data[MAX_DEQUE_SIZE];
    int front;
    int rear;
    int size;
} Deque;

// 덱 함수 선언
void init_deque(Deque *deque);
int is_deque_empty(Deque *deque);
int is_deque_full(Deque *deque);
void push_front(Deque *deque, int value);
void push_back(Deque *deque, int value);
int pop_front(Deque *deque);
int pop_back(Deque *deque);
int peek_front(Deque *deque);
int peek_back(Deque *deque);
void clear_deque(Deque *deque);
void display_deque(Deque *deque);

#endif // DEQUE_H
```

**src/deque.c**

```c
#include "deque.h"
#include <stdio.h>

void init_deque(Deque *deque) {
    deque->front = 0;
    deque->rear = -1;
    deque->size = 0;
}

int is_deque_empty(Deque *deque) {
    return deque->size == 0;
}

int is_deque_full(Deque *deque) {
    return deque->size == MAX_DEQUE_SIZE;
}

void push_front(Deque *deque, int value) {
    if (is_deque_full(deque)) {
        printf("Deque overflow\n");
        return;
    }
    deque->front = (deque->front - 1 + MAX_DEQUE_SIZE) % MAX_DEQUE_SIZE;
    deque->data[deque->front] = value;
    deque->size++;
}

void push_back(Deque *deque, int value) {
    if (is_deque_full(deque)) {
        printf("Deque overflow\n");
        return;
    }
    deque->rear = (deque->rear + 1) % MAX_DEQUE_SIZE;
    deque->data[deque->rear] = value;
    deque->size++;
}

int pop_front(Deque *deque) {
    if (is_deque_empty(deque)) {
        printf("Deque underflow\n");
        return -1;
    }
    int value = deque->data[deque->front];
    deque->front = (deque->front + 1) % MAX_DEQUE_SIZE;
    deque->size--;
    return value;
}

int pop_back(Deque *deque) {
    if (is_deque_empty(deque)) {
        printf("Deque underflow\n");
        return -1;
    }
    int value = deque->data[deque->rear];
    deque->rear = (deque->rear - 1 + MAX_DEQUE_SIZE) % MAX_DEQUE_SIZE;
    deque->size--;
    return value;
}

int peek_front(Deque *deque) {
    if (is_deque_empty(deque)) {
        printf("Deque is empty\n");
        return -1;
    }
    return deque->data[deque->front];
}

int peek_back(Deque *deque) {
    if (is_deque_empty(deque)) {
        printf("Deque is empty\n");
        return -1;
    }
    return deque->data[deque->rear];
}

void clear_deque(Deque *deque) {
    deque->front = 0;
    deque->rear = -1;
    deque->size = 0;
}

void display_deque(Deque *deque) {
    if (is_deque_empty(deque)) {
        printf("Deque is empty\n");
        return;
    }
    printf("Deque: ");
    int i = deque->front;
    for (int count = 0; count < deque->size; count++) {
        printf("%d ", deque->data[i]);
        i = (i + 1) % MAX_DEQUE_SIZE;
    }
    printf("\n");
}
```

**src/main.c**

```c
#include "stack.h"
#include "queue.h"
#include "linked_list.h"
#include "deque.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

Stack stack;
Queue queue;
Deque deque;
Node *list = NULL;

void process_command(char *command) {
    char cmd[50];
    int value;
    sscanf(command, "%s %d", cmd, &value);

    if (strcmp(cmd, "create") == 0) {
        if (strcmp(command, "create stack") == 0) {
            init_stack(&stack);
            printf("Stack created.\n");
        } else if (strcmp(command, "create queue") == 0) {
            init_queue(&queue);
            printf("Queue created.\n");
        } else if (strcmp(command, "create list") == 0) {
            list = NULL;
            printf("Linked list created.\n");
        } else if (strcmp(command, "create deque") == 0) {
            init_deque(&deque);
            printf("Deque created.\n");
        }
    } else if (strcmp(cmd, "push") == 0) {
        if (strstr(command, "push_front") == command) {
            push_front(&deque, value);
            printf("Pushed %d to front of deque.\n", value);
        } else if (strstr(command, "push_back") == command) {
            push_back(&deque, value);
            printf("Pushed %d to back of deque.\n", value);
        } else {
            push(&stack, value);
            printf("Pushed %d to stack.\n", value);
        }
    } else if (strcmp(cmd, "pop") == 0) {
        if (strstr(command, "pop_front") == command) {
            int popped = pop_front(&deque);
            if (popped != -1) {
                printf("Popped %d from front of deque.\n", popped);
            }
        } else if (strstr(command, "pop_back") == command) {
            int popped = pop_back(&deque);
            if (popped != -1) {
                printf("Popped %d from back of deque.\n", popped);
            }
        } else {
            int popped = pop(&stack);
            if (popped != -1) {
                printf("Popped %d from stack.\n", popped);
            }
        }
    } else if (strcmp(cmd, "peek") == 0) {
        if (strcmp(command, "peek stack") == 0) {
            int top = peek_stack(&stack);
            if (top != -1) {
                printf("Top element is %d.\n", top);
            }
        } else if (strcmp(command, "peek queue") == 0) {
            int front = peek_queue(&queue);
            if (front != -1) {
                printf("Front element is %d.\n", front);
            }
        } else if (strcmp(command, "peek_front deque") == 0) {
            int front = peek_front(&deque);
            if

 (front != -1) {
                printf("Front element is %d.\n", front);
            }
        } else if (strcmp(command, "peek_back deque") == 0) {
            int back = peek_back(&deque);
            if (back != -1) {
                printf("Back element is %d.\n", back);
            }
        }
    } else if (strcmp(cmd, "clear") == 0) {
        if (strcmp(command, "clear stack") == 0) {
            clear_stack(&stack);
            printf("Stack cleared.\n");
        } else if (strcmp(command, "clear queue") == 0) {
            clear_queue(&queue);
            printf("Queue cleared.\n");
        } else if (strcmp(command, "clear list") == 0) {
            clear_list(&list);
            printf("List cleared.\n");
        } else if (strcmp(command, "clear deque") == 0) {
            clear_deque(&deque);
            printf("Deque cleared.\n");
        }
    } else if (strcmp(cmd, "enqueue") == 0) {
        enqueue(&queue, value);
        printf("Enqueued %d to queue.\n", value);
    } else if (strcmp(cmd, "dequeue") == 0) {
        int dequeued = dequeue(&queue);
        if (dequeued != -1) {
            printf("Dequeued %d from queue.\n", dequeued);
        }
    } else if (strcmp(cmd, "insert_front") == 0) {
        insert_front(&list, value);
        printf("Inserted %d at front of list.\n", value);
    } else if (strcmp(cmd, "insert_end") == 0) {
        insert_end(&list, value);
        printf("Inserted %d at end of list.\n", value);
    } else if (strcmp(cmd, "delete") == 0) {
        delete_node(&list, value);
        printf("Deleted %d from list.\n", value);
    } else if (strcmp(cmd, "search") == 0) {
        Node *found = search_list(list, value);
        if (found) {
            printf("Found %d in list.\n", value);
        } else {
            printf("Did not find %d in list.\n", value);
        }
    } else if (strcmp(cmd, "display") == 0) {
        if (strcmp(command, "display stack") == 0) {
            display_stack(&stack);
        } else if (strcmp(command, "display queue") == 0) {
            display_queue(&queue);
        } else if (strcmp(command, "display list") == 0) {
            display_list(list);
        } else if (strcmp(command, "display deque") == 0) {
            display_deque(&deque);
        }
    } else {
        printf("Unknown command.\n");
    }
}

int main() {
    char command[100];
    printf("Data Structure Simulator\n");
    printf("Enter command: ");
    
    while (fgets(command, sizeof(command), stdin)) {
        // 개행 문자 제거
        command[strcspn(command, "\n")] = '\0';
        process_command(command);
        printf("Enter command: ");
    }
    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/stack.c src/queue.c src/linked_list.c src/deque.c
OBJS = $(SRCS:.c=.o)

TARGET = data_structure_simulator

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일**:
    - `deque.h`는 덱에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - 덱 함수로는 `init_deque`, `is_deque_empty`, `is_deque_full`, `push_front`, `push_back`, `pop_front`, `pop_back`, `peek_front`, `peek_back`, `clear_deque`, `display_deque`가 있습니다.

2. **소스 파일**:
    - `deque.c`는 덱 함수의 구현이 포함되어 있습니다.
    - `init_deque` 함수는 덱을 초기화합니다.
    - `is_deque_empty` 함수는 덱이 비어 있는지 확인합니다.
    - `is_deque_full` 함수는 덱이 가득 찼는지 확인합니다.
    - `push_front` 함수는 덱의 앞에 값을 추가합니다.
    - `push_back` 함수는 덱의 뒤에 값을 추가합니다.
    - `pop_front` 함수는 덱의 앞에서 값을 제거하고 반환합니다.
    - `pop_back` 함수는 덱의 뒤에서 값을 제거하고 반환합니다.
    - `peek_front` 함수는 덱의 앞의 값을 반환합니다.
    - `peek_back` 함수는 덱의 뒤의 값을 반환합니다.
    - `clear_deque` 함수는 덱을 초기화합니다.
    - `display_deque` 함수는 덱의 내용을 출력합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 사용자의 명령어를 처리합니다.
    - `process_command` 함수는 사용자가 입력한 명령어를 분석하고, 적절한 데이터 구조 연산을 수행합니다.
    - 스택, 큐, 링크드 리스트, 덱을 생성하고, 삽입, 삭제, 디스플레이 등의 연산을 수행할 수 있습니다.
    - 명령어에 따라 적절한 연산을 수행합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
