# Project 02

### 1.2 텍스트 기반 게임

#### 프로젝트 구조

```
text_game/
├── include/
│   └── game.h
├── src/
│   ├── game.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 텍스트 기반 게임

이 프로젝트는 간단한 텍스트 기반의 RPG 게임입니다. 플레이어는 몬스터와 전투를 벌이고, 승리하거나 패배할 때까지 게임이 진행됩니다.

## 파일 구조

- `include/game.h` : 게임 함수와 구조체 선언이 포함된 헤더 파일.
- `src/game.c` : 게임 함수와 구조체 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./text_game
```

## 게임 설명

게임이 시작되면 플레이어와 몬스터의 초기 체력이 설정됩니다. 플레이어는 공격을 선택하여 몬스터를 공격할 수 있습니다. 몬스터는 무작위로 반격합니다. 플레이어와 몬스터 중 하나의 체력이 0 이하가 되면 게임이 종료됩니다.
```

#### 코드 구현 및 상세 설명

**include/game.h**

```c
#ifndef GAME_H
#define GAME_H

// 플레이어와 몬스터의 상태를 저장하는 구조체
typedef struct {
    char *name;
    int health;
    int attack_power;
} Character;

// 함수 선언
void attack(Character *attacker, Character *defender);
void print_status(Character *player, Character *monster);
void play_game(Character *player, Character *monster);

#endif // GAME_H
```

**src/game.c**

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include "game.h"

// 공격 함수
void attack(Character *attacker, Character *defender) {
    int damage = rand() % attacker->attack_power;
    defender->health -= damage;
    printf("%s attacks %s for %d damage!\n", attacker->name, defender->name, damage);
}

// 현재 상태를 출력하는 함수
void print_status(Character *player, Character *monster) {
    printf("\nStatus:\n");
    printf("%s: %d HP\n", player->name, player->health);
    printf("%s: %d HP\n", monster->name, monster->health);
}

// 게임 플레이 함수
void play_game(Character *player, Character *monster) {
    srand(time(NULL));
    
    while (player->health > 0 && monster->health > 0) {
        print_status(player, monster);

        printf("\nChoose an action:\n1. Attack\n2. Quit\n");
        int choice;
        scanf("%d", &choice);

        if (choice == 1) {
            attack(player, monster);
            if (monster->health > 0) {
                attack(monster, player);
            }
        } else {
            printf("You quit the game.\n");
            break;
        }

        if (player->health <= 0) {
            printf("You have been defeated!\n");
        } else if (monster->health <= 0) {
            printf("You have defeated the monster!\n");
        }
    }
}
```

**src/main.c**

```c
#include "game.h"

int main() {
    Character player = {"Player", 100, 20};
    Character monster = {"Monster", 80, 15};

    printf("Welcome to the Text-based RPG Game!\n");
    play_game(&player, &monster);

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/game.c
OBJS = $(SRCS:.c=.o)

TARGET = text_game

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`game.h`)**:
    - 게임에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - `Character` 구조체는 플레이어와 몬스터의 상태를 저장합니다.
    - `attack`, `print_status`, `play_game` 함수의 선언이 포함되어 있습니다.

2. **소스 파일(`game.c`)**:
    - 게임의 주요 기능이 구현되어 있습니다.
    - `attack` 함수는 공격을 수행하여 상대방의 체력을 감소시킵니다.
    - `print_status` 함수는 플레이어와 몬스터의 현재 상태를 출력합니다.
    - `play_game` 함수는 게임의 메인 루프를 담당합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점입니다.
    - 플레이어와 몬스터의 초기 상태를 설정하고, `play_game` 함수를 호출하여 게임을 시작합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.

### 프로젝트 설명

이 텍스트 기반 게임 프로젝트는 간단한 RPG 게임을 구현합니다. 플레이어는 공격을 선택하여 몬스터와 싸우며, 몬스터는 무작위로 반격합니다. 플레이어와 몬스터 중 하나의 체력이 0 이하가 되면 게임이 종료됩니다.

- 플레이어는 공격을 선택할 수 있습니다.
- 몬스터는 무작위로 반격합니다.
- 플레이어와 몬스터의 체력 상태를 출력합니다.
- 게임이 종료되면 승리 또는 패배 메시지를 출력합니다.

---

### 1.2 텍스트 기반 게임

#### 프로젝트 구조

```
text_adventure_game/
├── include/
│   └── game.h
├── src/
│   ├── game.c
│   └── main.c
├── Makefile
└── README.md
```

#### README.md

```markdown
# 텍스트 기반 게임

이 프로젝트는 간단한 텍스트 기반 어드벤처 게임입니다. 플레이어는 다양한 명령을 통해 게임을 진행하며, 목표를 달성합니다.

## 파일 구조

- `include/game.h` : 게임 함수와 구조체 선언이 포함된 헤더 파일.
- `src/game.c` : 게임 함수의 구현이 포함된 소스 파일.
- `src/main.c` : 프로그램의 진입점인 main 함수가 포함된 소스 파일.
- `Makefile` : 프로젝트 빌드를 위한 Makefile.

## 빌드 방법

```sh
make
```

## 실행 방법

```sh
./text_adventure_game
```

## 게임 명령어

- `look` : 현재 위치의 설명을 출력합니다.
- `go <direction>` : 주어진 방향으로 이동합니다 (예: go north).
- `help` : 사용 가능한 명령어 목록을 출력합니다.
- `quit` : 게임을 종료합니다.
```

#### 코드 구현 및 상세 설명

**include/game.h**

```c
#ifndef GAME_H
#define GAME_H

#define MAX_DESC 256
#define MAX_ROOMS 5

// 방향 열거형
typedef enum {
    NORTH,
    SOUTH,
    EAST,
    WEST,
    DIR_COUNT
} Direction;

// 방 구조체
typedef struct {
    char description[MAX_DESC];
    int exits[DIR_COUNT];
} Room;

// 게임 초기화 함수
void initialize_game(Room rooms[MAX_ROOMS]);

// 명령 처리 함수
void process_command(char *command, Room rooms[MAX_ROOMS], int *current_room);

// 명령어 도움말 출력 함수
void print_help();

// 방 설명 출력 함수
void look(Room rooms[MAX_ROOMS], int current_room);

// 이동 함수
void go(Room rooms[MAX_ROOMS], int *current_room, Direction direction);

#endif // GAME_H
```

**src/game.c**

```c
#include "game.h"
#include <stdio.h>
#include <string.h>

// 방 초기화 함수
void initialize_game(Room rooms[MAX_ROOMS]) {
    // 방 0 (시작 방)
    strcpy(rooms[0].description, "You are in a small room. There are doors to the north and east.");
    rooms[0].exits[NORTH] = 1;
    rooms[0].exits[EAST] = 2;
    rooms[0].exits[SOUTH] = -1;
    rooms[0].exits[WEST] = -1;

    // 방 1
    strcpy(rooms[1].description, "You are in a dark corridor. There is a door to the south.");
    rooms[1].exits[NORTH] = -1;
    rooms[1].exits[SOUTH] = 0;
    rooms[1].exits[EAST] = -1;
    rooms[1].exits[WEST] = -1;

    // 방 2
    strcpy(rooms[2].description, "You are in a bright hall. There are doors to the west and north.");
    rooms[2].exits[NORTH] = 3;
    rooms[2].exits[SOUTH] = -1;
    rooms[2].exits[EAST] = -1;
    rooms[2].exits[WEST] = 0;

    // 방 3
    strcpy(rooms[3].description, "You are in a library. There is a door to the south.");
    rooms[3].exits[NORTH] = -1;
    rooms[3].exits[SOUTH] = 2;
    rooms[3].exits[EAST] = -1;
    rooms[3].exits[WEST] = -1;

    // 방 4 (승리 방)
    strcpy(rooms[4].description, "You have found the treasure room! Congratulations, you win!");
    rooms[4].exits[NORTH] = -1;
    rooms[4].exits[SOUTH] = -1;
    rooms[4].exits[EAST] = -1;
    rooms[4].exits[WEST] = -1;
}

// 명령어 도움말 출력 함수
void print_help() {
    printf("Available commands:\n");
    printf("  look       - Look around the current room\n");
    printf("  go <dir>   - Move in the specified direction (north, south, east, west)\n");
    printf("  help       - Show this help message\n");
    printf("  quit       - Quit the game\n");
}

// 방 설명 출력 함수
void look(Room rooms[MAX_ROOMS], int current_room) {
    printf("%s\n", rooms[current_room].description);
}

// 이동 함수
void go(Room rooms[MAX_ROOMS], int *current_room, Direction direction) {
    int next_room = rooms[*current_room].exits[direction];
    if (next_room == -1) {
        printf("You can't go that way.\n");
    } else {
        *current_room = next_room;
        look(rooms, *current_room);
    }
}

// 명령 처리 함수
void process_command(char *command, Room rooms[MAX_ROOMS], int *current_room) {
    if (strcmp(command, "look") == 0) {
        look(rooms, *current_room);
    } else if (strncmp(command, "go ", 3) == 0) {
        char *direction_str = command + 3;
        Direction direction;
        if (strcmp(direction_str, "north") == 0) {
            direction = NORTH;
        } else if (strcmp(direction_str, "south") == 0) {
            direction = SOUTH;
        } else if (strcmp(direction_str, "east") == 0) {
            direction = EAST;
        } else if (strcmp(direction_str, "west") == 0) {
            direction = WEST;
        } else {
            printf("Unknown direction: %s\n", direction_str);
            return;
        }
        go(rooms, current_room, direction);
    } else if (strcmp(command, "help") == 0) {
        print_help();
    } else if (strcmp(command, "quit") == 0) {
        printf("Thank you for playing!\n");
        exit(0);
    } else {
        printf("Unknown command: %s\n", command);
        print_help();
    }
}
```

**src/main.c**

```c
#include "game.h"
#include <stdio.h>
#include <stdlib.h>

int main() {
    Room rooms[MAX_ROOMS];
    int current_room = 0;
    char command[256];

    initialize_game(rooms);
    printf("Welcome to the text adventure game!\n");
    look(rooms, current_room);

    while (1) {
        printf("> ");
        if (fgets(command, sizeof(command), stdin) != NULL) {
            command[strcspn(command, "\n")] = 0; // 개행 문자 제거
            process_command(command, rooms, &current_room);
        }
    }

    return 0;
}
```

**Makefile**

```makefile
CC = gcc
CFLAGS = -Iinclude -Wall

SRCS = src/main.c src/game.c
OBJS = $(SRCS:.c=.o)

TARGET = text_adventure_game

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

clean:
    rm -f $(TARGET) $(OBJS)
```

### 코드 설명

1. **헤더 파일(`game.h`)**:
    - 게임에 필요한 구조체와 함수의 선언이 포함되어 있습니다.
    - `Room` 구조체는 각 방의 설명과 출구를 정의합니다.
    - 방향(Direction)을 나타내는 열거형이 정의되어 있습니다.

2. **소스 파일(`game.c`)**:
    - 게임 초기화 함수(`initialize_game`)는 방의 설명과 출구를 설정합니다.
    - 명령어 처리 함수(`process_command`)는 사용자의 명령을 처리합니다.
    - `look` 함수는 현재 방의 설명을 출력합니다.
    - `go` 함수는 사용자가 이동하려는 방향으로 이동을 시도하고, 이동 가능 여부를 판단하여 이동합니다.
    - `print_help` 함수는 사용 가능한 명령어를 출력합니다.

3. **메인 파일(`main.c`)**:
    - 프로그램의 진입점으로, 게임을 초기화하고, 사용자로부터 명령을 입력받아 처리합니다.
    - `fgets`를 사용하여 사용자 입력을 받고, `process_command` 함수에 전달합니다.

4. **Makefile**:
    - `gcc` 컴파일러를 사용하여 프로젝트를 빌드합니다.
    - `CFLAGS`에 `include` 디렉토리를 포함시킵니다.
    - 소스 파일을 컴파일하여 오브젝트 파일을 생성하고, 이를 링크하여 실행 파일을 만듭니다.
    - `clean` 타겟을 정의하여 생성된 파일들을 삭제합니다.
