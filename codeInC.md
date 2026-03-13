
# overview

1. module : header(_.h)와 source(_.c)
2. 선언과 정의, 그리고 scope
3. header
4. source 코드의 구성
5. naming
6. appendix 

* must read 1~5


# 1. module

```
|----------------|
| Module         |
|   * data       |==
|   * interface  |==
|----------------|
```

* 무엇을 공개 것인가?
* "상태"문제
*
> 모듈의 분할은 설계의 첫단계

## C 언어에서 사용가능한 모듈화 방법

1. 헤더/소스 파일 분리
2. naming convention이 생각보다 중요
3. 전형적 기법들



# 2. 선언과 정의, 그리고 scope

* 3~5를 위한 최소한의 지식

```c++
// test.c

int a;                        // ?
extern int b;                 // X
static int c;

void f( int d, const int e)
{
    int h;
    static int i;            // X
    extern int j;            // X
}

static void g() {             // static 

}
```

* declaration ( p.465 in "C Programming: A Modern Approach, 2nd ed")
> 1. storage
> 2. scope
> 3. linkage

| name | storage  | scope | linkage   |
|------|----------|-------|-----------|
| a    | static   | file  | external  |
| b    | static   | file  | use other |
| c    | static   | file  | internal  |
| d    | auto     | block | -         |
| e    | auto     | block | -         |
| f    | function | -     | external  |
| g    | function | -     | interanl  |
| h    | auto     | block | -         |
| i    | static   | block | -         |
| j    | static   | block | use other |


# 3. header
## include 구성순서
```c++

/* ========================================================================
 * 파일 설명 및 저작권 정보
 * database.h - Database connection and query management interface
 * Copyright (c) 2024 Your Company
 * ======================================================================== */

// 1. Header Guard (또는 #pragma once)
#ifndef DATABASE_H
#define DATABASE_H

// 2. 필수 시스템 헤더만 포함 :: inlude는 최소화
#include <stddef.h>        // size_t
#include <stdbool.h>       // bool

// 3. 조건부 컴파일
#ifdef __cplusplus
extern "C" {
#endif

// 4. 매크로 정의
#define DB_MAX_HOST_LEN         256
#define DB_DEFAULT_PORT         5432
#define DB_VERSION_MAJOR        2
#define DB_VERSION_MINOR        1

// 5. 타입 정의
// 5-1. 전방 선언
struct database_connection;
struct query_result;

// 5-2. enum 정의
typedef enum {
    DB_SUCCESS = 0,
    DB_ERROR_CONNECTION = -1,
    DB_ERROR_TIMEOUT = -2,
    DB_ERROR_INVALID_QUERY = -3
} db_error_t;

typedef enum {
    DB_TYPE_ORACLE,
    DB_TYPE_MYSQL,
    DB_TYPE_POSTGRESQL
} db_type_t;

// 5-3. 구조체 정의 (public만)
typedef struct {
    int major;
    int minor;
    int patch;
} db_version_t;

typedef struct {
    char host[DB_MAX_HOST_LEN];
    int port;
    db_type_t type;
} db_config_t;

// 5-4. 함수 포인터 타입
typedef int (*db_callback_t)(const char* data, size_t len, void* userdata);
typedef void (*db_error_handler_t)(db_error_t error, const char* message);

// 6. 외부 변수 선언 (extern) --> 해당 소스
extern const char* db_version_string;
extern db_error_handler_t db_global_error_handler;

// 7. 함수 선언
// 7-1. 초기화/정리 함수
int db_init(const db_config_t* config);
void db_cleanup(void);

// 7-2. 연결 관리 함수
struct database_connection* db_connect(const char* host, int port);
void db_disconnect(struct database_connection* conn);
bool db_is_connected(struct database_connection* conn);

// 7-3. 쿼리 함수
struct query_result* db_execute_query(struct database_connection* conn, 
                                     const char* sql);
int db_execute_async(struct database_connection* conn, 
                     const char* sql, 
                     db_callback_t callback,
                     void* userdata);

// 7-4. 결과 처리 함수
size_t db_result_row_count(struct query_result* result);
const char* db_result_get_string(struct query_result* result, 
                                int row, int col);
void db_result_free(struct query_result* result);

// 7-5. 유틸리티 함수
const char* db_error_string(db_error_t error);
db_version_t db_get_version(void);

// 8. 인라인 함수 (필요한 경우)
static inline bool db_version_is_compatible(const db_version_t* version) {
    return (version->major == DB_VERSION_MAJOR && 
            version->minor <= DB_VERSION_MINOR);
}

// 9. C++ 가드 종료
#ifdef __cplusplus
}
#endif

// 10. Header Guard 종료
#endif /* DATABASE_H */
```

# 4. source
## source code의 구성순서

```c++ 
/* ========================================================================
 * 파일 설명 및 저작권 정보
 * ======================================================================== */

// 1. 헤더 파일들
#include "mymodule.h"      // 자신의 헤더 먼저

#include <stdio.h>         // 시스템 헤더들 (알파벳순)
#include <stdlib.h>
#include <string.h>

#include "config.h"        // 프로젝트 헤더들 (알파벳순)
#include "logger.h"
#include "utils.h"

// 2. 매크로 정의
#define MAX_BUFFER_SIZE 1024
#define DEFAULT_TIMEOUT 30

// 3. 타입 정의 (구조체, enum, typedef)
typedef enum {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_ERROR
} module_state_t;

struct internal_data {
    int value;
    char buffer[MAX_BUFFER_SIZE];
};

// 4. 전역 변수 (static 우선)
static int            g_connection_count = 0;
static module_state_t g_current_state = STATE_IDLE;

int                   g_public_counter = 0;  // 외부에서 접근 가능한 전역변수

// 5. Static 함수 전방 선언
static int validate_input(const char* input);
static void cleanup_resources(void);
static bool connect_to_server(const char* host, int port);

// 6. Public 함수 구현 (헤더에 선언된 함수들)
int public_function(const char* data) {
    // 구현...
}

// 7. Static 함수 구현
static int validate_input(const char* input) {
    // 구현...
}

static void cleanup_resources(void) {
    // 구현...
}
```

## 모듈 분할 사례

* Data중심 분할

```c++
// person.h
#ifndef PERSON_H
#define PERSON_H

typedef struct {
    char name[64];
    int age;
    char email[128];
} person_t;

// person만 다루는 함수들
person_t* person_create(const char* name, int age);
void person_destroy(person_t* person);
void person_set_email(person_t* person, const char* email);
const char* person_get_name(const person_t* person);
void person_print(const person_t* person);

#endif
```

```c++
// person.h
#ifndef PERSON_H
#define PERSON_H

typedef struct {
    char name[64];
    int age;
    char email[128];
} person_t;

// person만 다루는 함수들
person_t* person_create(const char* name, int age);
void person_destroy(person_t* person);
void person_set_email(person_t* person, const char* email);
const char* person_get_name(const person_t* person);
void person_print(const person_t* person);

#endif
```

* action with some-type
```c++
// geometry.h
#ifndef GEOMETRY_H
#define GEOMETRY_H

#include "point.h"
#include "circle.h"

// 각 타입의 기본 함수들
point_t point_create(double x, double y);
double point_distance(const point_t* p1, const point_t* p2);
void point_print(const point_t* point);

circle_t circle_create(point_t center, double radius);
double circle_area(const circle_t* circle);
double circle_circumference(const circle_t* circle);

// 두 타입을 함께 사용하는 함수들 (자연스럽게 같은 모듈에)
bool point_in_circle(const point_t* point, const circle_t* circle);
point_t circle_point_at_angle(const circle_t* circle, double angle);
bool circles_intersect(const circle_t* c1, const circle_t* c2);

#endif
```

* more
```c++
#ifndef SHAPE_H
#define SHAPE_H

typedef enum {
    SHAPE_RECTANGLE,
    SHAPE_CIRCLE,
    SHAPE_TRIANGLE
} shape_type_t;

typedef struct shape shape_t;

// 가상 함수 테이블
typedef struct {
    void (*draw)(const shape_t* shape);
    double (*area)(const shape_t* shape);
    void (*move)(shape_t* shape, int dx, int dy);
    void (*destroy)(shape_t* shape);
} shape_vtable_t;

// 기본 shape 구조
struct shape {
    shape_type_t type;
    position_t position;
    const shape_vtable_t* vtable;
};
#endif
```

```c++
#ifndef GRAPHICS_BASE_H
#define GRAPHICS_BASE_H

#include "position.h"
#include "shape.h"

void shape_draw(const shape_t* shape);
double shape_area(const shape_t* shape);
void shape_move(shape_t* shape, int dx, int dy);
void shape_destroy(shape_t* shape);

#endif
```

* sample practice

```
// 각 타입별 기본 모듈
person.h/c      // person_t 관련 기본 함수들
product.h/c     // product_t 관련 기본 함수들

//  관계/상호작용 모듈  
transaction.h/c // person + product 상호작용
inventory.h/c   // product 컬렉션 관리
customer.h/c    // person 확장 기능

//  유틸리티/헬퍼 모듈
business_logic.h/c  // 복잡한 비즈니스 로직
validation.h/c      // 검증 함수들
conversion.h/c      // 타입 간 변환

// 시스템/매니저 모듈
app_manager.h/c     // 전체 시스템 관리
database_layer.h/c  // 영속성 계층
```
* Some principle
1. **단일 타입 함수** → 해당 타입 모듈
2. **다중 타입 함수** → 관계 전담 모듈 또는 상위 모듈
3. **변환 함수** → 대상 타입 모듈 또는 변환 전담 모듈
4. **유틸리티 함수** → 별도 유틸리티 모듈
5. **시스템 함수** → 시스템 관리 모듈

* 고려사항
> - **의존성 방향**: 순환 참조 방지
> - **응집도**: 관련 기능끼리 그룹화
> - **결합도**: 모듈 간 결합 최소화
> - **재사용성**: 함수의 재사용 가능성
> - **테스트 용이성**: 단위 테스트 작성 편의성

    : 가장 중요한 것은 **일관성**과 **논리적 구조**


# 5. Naming

## naming convention

### namespace 부재

```c++
// logger.h
#ifndef LOGGER_H
#define LOGGER_H

typedef enum {
    LOGGER_LEVEL_DEBUG,
    LOGGER_LEVEL_INFO,
    LOGGER_LEVEL_ERROR
} logger_level_t;

// 모든 함수에 logger_ 접두사
int  logger_init(const char* filename);
void logger_cleanup(void);
void logger_set_level(logger_level_t level);
void logger_write(logger_level_t level, const char* format, ...);
void logger_debug(const char* format, ...);
void logger_info(const char* format, ...);
void logger_error(const char* format, ...);
#endif
```

### variable scope & life

```c++
// globals.h
#ifndef GLOBALS_H
#define GLOBALS_H

// 전역 변수는 g_ 접두사
extern int g_debug_level;
extern char* g_config_file_path;
extern bool g_is_initialized;
extern const char* g_version_string;

// 전역 상수는 대문자 + 모듈명
#define LOG_MAX_MESSAGE_SIZE 1024
#define DB_CONNECTION_TIMEOUT 30

#endif
```

```c++
// database.c
#include "database.h"

// 파일 내부 전역 변수는 s_ 접두사
static int s_connection_count = 0;
static bool s_database_initialized = false;
static char s_last_error_message[512];
static struct db_connection* s_active_connections[MAX_CONNECTIONS];

// 파일 내부 상수는 k_ 접두사 
static const int   k_max_retry_count = 3;
static const char* k_default_database_name = "default.db";
```

### typedef is not type-definition.

```c++
// header

// typedef _t 접미사
typedef struct {
    char db_host[256];
    int  db_port;
    char db_name[64];
    
    char ui_theme[32];
    int  ui_window_width;
    int  ui_window_height;
    
    bool net_ssl_enabled;
    int  net_timeout;
} application_config_t;

typedef void (*event_handler_t)(int event_type, void* data);
typedef bool (*validation_callback_t)(const char* input);
typedef int (*comparison_func_t)(const void* a, const void* b);

typedef struct {
    char name[64];
    
    // 함수 포인터는 동사로 시작
    int (*initialize)(void);
    void (*cleanup)(void);
    bool (*process)(const char* data);
    void (*on_error)(int error_code);
} plugin_interface_t;
```

```c++
// event_system.c
#include "event_system.h"

// 함수 포인터 저장을 위한 static 변수
static event_handler_t s_registered_handlers[10];       // <<<<
static int s_handler_count = 0;

int event_register_handler(event_handler_t handler) {
    int available_slot = -1;
    bool registration_success = false;
    
    for (int i = 0; i < 10; i++) {
        if (s_registered_handlers[i] == NULL) {          // <<<<
            available_slot = i;                   
            break;
        }
    }
    
    if (available_slot >= 0) {
        s_registered_handlers[available_slot] = handler;   // <<<<
        registration_success = true;                       
        s_handler_count++;                                 // <<<<
    }
    
    return registration_success ? 0 : -1;
}
```

# Appendix

### Tip : Compiler 최적화를 통한 stack-buffer의 번거로움 해소

```c++
// 여러분의 코드
// out parameter 방식
// -------------------------------------------------------
#include <stdio.h>

// function: caller가 준비한 버퍼에 직접 채움
void f(char c, char* out, size_t out_sz) {
    for (size_t i = 0; i < out_sz; i++) {
        out[i] = c;
    }
}

// caller2: 단순히 f를 호출 
void caller2(char c, char* out, size_t out_sz) {
    f(c, out, out_sz);
}

// caller1: 최종 버퍼 준비
int main() {
    char buf[1024];                // <<<<<< 
    caller2('A', buf, 1024);

    printf("First: %c, Last: %c\n", buf[0], buf[1023]);
    return 0;
}
```

* compiler 최적화를 공부한 사람의 코드

```c++
// 컴파일러 최적화를 이용하는 방식 : 동일한 성능수준으로 최적화됨.
// RVO(Return Value Optimization)/NRVO(Named Return Value Optimization)부름.
// (C++에서 쓰이는 용어지만 C에서도 같은 개념으로 이해하면 됨.)
// c99이상 지원하는 컴파일러(gcc, clang, msvc 등)에서 제공됨. ( -O2, -O3 )
// 주의) 최적화를 off한다면, 복사비용이 호출횟수만큼 발생
// -------------------------------------------------------
#include <stdio.h>

// 단, automatic storage사용할 것이므로, 고정size써야하는 건 마찬가지.
typedef struct {
    char buf[1024];  
} StringBuf;

// function: 큰 struct 반환 :: 인자를 봐라.
StringBuf f(char c) {
    StringBuf s;
    for (int i = 0; i < 1024; i++) {
        s.buf[i] = c;
    }
    return s; // ABI 규칙에 따라 caller 버퍼에 직접 작성됨
}

// caller2: f를 호출해서 struct 반환 :: 인자를 봐라.
StringBuf caller2(char c) {
    return f(c);
}

// caller1: 최종 결과 받음
int main() {
    StringBuf result = caller2('B');

    printf("First: %c, Last: %c\n", result.buf[0], result.buf[1023]);
    return 0;
}
```
* 주의(당연한 얘기)
> - 말그대로 불필요한 임시변수 사용을 줄이기 위한 것.
> - char buf[1024]를 여러개 만드는 만큼의 stack을 소비한다.
> - 생각없이 사용하면 stack-overflow 발생할 수 있다.
    >  (buf를 너무 많이 만들면 overflow나듯이)
