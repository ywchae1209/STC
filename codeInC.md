
# basic 

1. module : header(_.h)와 source(_.c) 
2. 선언과 정의, 그리고 scope
3. header
4. source 코드의 구성
5. naming
6. c++이 되고 싶었던 c :: for developer only
6. appendix :: for developer only

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

int a;
extern int b;
static int c;

// declaration
void f( int d, const int e)   // definition
{
    int h;
    static int i;
    extern int j;
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
static int g_connection_count = 0;
static module_state_t g_current_state = STATE_IDLE;

int g_public_counter = 0;  // 외부에서 접근 가능한 전역변수

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
int logger_init(const char* filename);
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
    int db_port;
    char db_name[64];
    
    char ui_theme[32];
    int ui_window_width;
    int ui_window_height;
    
    bool net_ssl_enabled;
    int net_timeout;
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

# 6. c++이 되고 싶었던 c : Part-One

language feature가 빈약한 C언어에서 선택할 수 밖에 없었던 방법들.

## generic is not-generic

* generic 안전장치가 빈약하므로, 매우 주의깊게 사용해야 함.
  ( 반드시 필요한 곳 아니면, 사용하지 않는 것도 하나의 방편.)

```c++
// 함수형 매크로는 소문자로 시작 (함수와 구분)
#define config_min(a, b)         ((a) < (b) ? (a) : (b))
#define config_max(a, b)         ((a) > (b) ? (a) : (b))


////////////////////////////////////////////////////////////////////////////////

/* C99까지는 매크로에서 타입 구분 불가능 */
#define PRINT_VALUE(x) printf("%d\n", (int)(x))  /* 강제 캐스팅 필요 */

/* 타입별로 별도 매크로 정의해야 함 */
#define PRINT_INT(x)    printf("%d\n", x)
#define PRINT_FLOAT(x)  printf("%f\n", x)
#define PRINT_STRING(x) printf("%s\n", x)


////////////////////////////////////////////////////////////////////////////////
/* C11: _Generic으로 컴파일 타임 타입 선택 가능 */
#define PRINT_VALUE(x) _Generic((x), \
    int: printf("%d\n", x), \
    float: printf("%f\n", x), \
    double: printf("%lf\n", x), \
    char*: printf("%s\n", x), \
    default: printf("Unknown type\n") \
)

/* C11 ---------------- */
#define safe_alloc(ptr, count) _Generic((ptr), \
    int*: (ptr) = malloc(sizeof(int) * (count)), \
    float*: (ptr) = malloc(sizeof(float) * (count)), \
    double*: (ptr) = malloc(sizeof(double) * (count)), \
    char*: (ptr) = malloc(sizeof(char) * (count)), \
    default: (ptr) = malloc(sizeof(*(ptr)) * (count)) \
)

/* C11 ---------------- */
#define safe_copy(dest, src, count) _Generic((dest), \
    int*: memcpy((dest), (src), sizeof(int) * (count)), \
    float*: memcpy((dest), (src), sizeof(float) * (count)), \
    double*: memcpy((dest), (src), sizeof(double) * (count)), \
    char*: memcpy((dest), (src), sizeof(char) * (count)), \
    default: memcpy((dest), (src), sizeof(*(dest)) * (count)) \
)
```

## emulate OOP by handler( why so many handles.) :: 권장

```c++
// file_handle.h
#ifndef FILE_HANDLE_H
#define FILE_HANDLE_H

#include <stddef.h>
#include <stdbool.h>

// 불투명 포인터 - 구현 완전히 숨김
typedef struct file_handle file_handle_t;

// 공개 인터페이스만 노출
file_handle_t* file_open(const char* filename, const char* mode);
void file_close(file_handle_t* handle);
size_t file_read(file_handle_t* handle, void* buffer, size_t size);
size_t file_write(file_handle_t* handle, const void* data, size_t size);
bool file_seek(file_handle_t* handle, long offset, int whence);
long file_tell(file_handle_t* handle);
bool file_eof(file_handle_t* handle);
const char* file_get_error(file_handle_t* handle);

#endif
```

```c++
// file_handle.c
#include "file_handle.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 실제 구조체 정의 - 외부에서 접근 불가
struct file_handle {
    FILE* fp;
    char filename[256];
    char mode[8];
    char last_error[512];
    size_t bytes_read;
    size_t bytes_written;
    bool is_open;
};

file_handle_t* file_open(const char* filename, const char* mode) {
    if (!filename || !mode) return NULL;
    
    file_handle_t* handle = malloc(sizeof(file_handle_t));
    if (!handle) return NULL;
    
    // 내부 구조체 초기화
    handle->fp = fopen(filename, mode);
    if (!handle->fp) {
        snprintf(handle->last_error, sizeof(handle->last_error),
                "Cannot open file: %s", filename);
        free(handle);
        return NULL;
    }
    
    strncpy(handle->filename, filename, sizeof(handle->filename) - 1);
    strncpy(handle->mode, mode, sizeof(handle->mode) - 1);
    handle->bytes_read = 0;
    handle->bytes_written = 0;
    handle->is_open = true;
    handle->last_error[0] = '\0';
    
    return handle;
}

void file_close(file_handle_t* handle) {
    if (!handle) return;
    
    if (handle->fp) {
        fclose(handle->fp);
        handle->fp = NULL;
    }
    handle->is_open = false;
    free(handle);
}

size_t file_read(file_handle_t* handle, void* buffer, size_t size) {
    if (!handle || !handle->is_open || !buffer) return 0;
    
    size_t bytes_read = fread(buffer, 1, size, handle->fp);
    handle->bytes_read += bytes_read;
    
    if (bytes_read == 0 && ferror(handle->fp)) {
        snprintf(handle->last_error, sizeof(handle->last_error),
                "Read error occurred");
    }
    
    return bytes_read;
}

size_t file_write(file_handle_t* handle, const void* data, size_t size) {
    if (!handle || !handle->is_open || !data) return 0;
    
    size_t bytes_written = fwrite(data, 1, size, handle->fp);
    handle->bytes_written += bytes_written;
    
    if (bytes_written != size) {
        snprintf(handle->last_error, sizeof(handle->last_error),
                "Write error occurred");
    }
    
    return bytes_written;
}

const char* file_get_error(file_handle_t* handle) {
    if (!handle) return "Invalid handle";
    return handle->last_error;
}
```

* why? 
1. 캡슐화 (구현은 공개되지 않음 )
2. 구현체가 바뀌어도, client코드는 재컴파일 불필요(라이브러리 업데이트 binary호환성)
3. 타입안정성( 다른 타입 포인터와 혼용불가, 컴파일타임 체크)


## co-product type in C  using struct. : A or B or C :: 많이 사용되는 기법

* Element

1. **Tag enum**: 어떤 variant인지 표시
2. **Union**: 실제 데이터 저장
3. **구조체**: Tag + Union을 묶음

* Safe implementation
> - 항상 tag를 먼저 확인
> - switch문으로 모든 케이스 처리
> - 생성자 함수로 올바른 초기화



### 1. basic

```c++
// result.h
#ifndef RESULT_H
#define RESULT_H

#include <stdbool.h>

// Sum Type: Result<T, E> 스타일
typedef enum {
    RESULT_OK,
    RESULT_ERROR
} result_tag_t;

typedef struct {
    result_tag_t tag;       // <<<<<< 
    union {
        int value;          // Success 값
        const char* error;  // Error 메시지
    } data;
} result_t;

// 생성자 함수들
result_t result_ok(int value);
result_t result_error(const char* message);

// 패턴 매칭 함수들
bool result_is_ok(const result_t* result);
bool result_is_error(const result_t* result);
int result_unwrap(const result_t* result);
const char* result_get_error(const result_t* result);

#endif
```
```c++
// result.c
#include "result.h"
#include <stdio.h>
#include <stdlib.h>

result_t result_ok(int value) {
    return (result_t) {
        .tag = RESULT_OK,
        .data.value = value
    };
}

result_t result_error(const char* message) {
    return (result_t) {
        .tag = RESULT_ERROR,
        .data.error = message
    };
}

bool result_is_ok(const result_t* result) {
    return result && result->tag == RESULT_OK;
}

int result_unwrap(const result_t* result) {
    if (result->tag != RESULT_OK) {
        fprintf(stderr, "Error: Tried to unwrap error result\n");
        exit(1);
    }
    return result->data.value;
}

const char* result_get_error(const result_t* result) {
    return (result->tag == RESULT_ERROR) ? result->data.error : NULL;
}
```

* usage

```c++
// main.c
#include "result.h"
#include <stdio.h>

result_t divide(int a, int b) {
    if (b == 0) {
        return result_error("Division by zero");
    }
    return result_ok(a / b);
}

int main(void) {
    // 성공 케이스
    result_t res1 = divide(10, 2);
    if (result_is_ok(&res1)) {
        printf("Result: %d\n", result_unwrap(&res1));
    }
    
    // 에러 케이스  
    result_t res2 = divide(10, 0);
    if (result_is_error(&res2)) {
        printf("Error: %s\n", result_get_error(&res2));
    }
    
    return 0;
}
```
### 2. little complex( but, still simple )

```c++
// shape.h
#ifndef SHAPE_H
#define SHAPE_H

// Shape Sum Type (Circle | Rectangle | Triangle)
typedef enum {
    SHAPE_CIRCLE,
    SHAPE_RECTANGLE, 
    SHAPE_TRIANGLE
} shape_type_t;

typedef struct {
    double radius;
} circle_data_t;

typedef struct {
    double width;
    double height;
} rectangle_data_t;

typedef struct {
    double base;
    double height;
} triangle_data_t;

typedef struct {
    shape_type_t type;
    union {
        circle_data_t circle;
        rectangle_data_t rectangle;
        triangle_data_t triangle;
    } data;
} shape_t;

// 생성자들
shape_t shape_circle(double radius);
shape_t shape_rectangle(double width, double height);
shape_t shape_triangle(double base, double height);

// 패턴 매칭으로 면적 계산
double shape_area(const shape_t* shape);
void shape_print(const shape_t* shape);

#endif
```

```c++
// shape.c
#include "shape.h"
#include <stdio.h>
#include <math.h>

shape_t shape_circle(double radius) {
    return (shape_t) {
        .type = SHAPE_CIRCLE,
        .data.circle = { .radius = radius }
    };
}

shape_t shape_rectangle(double width, double height) {
    return (shape_t) {
        .type = SHAPE_RECTANGLE,
        .data.rectangle = { .width = width, .height = height }
    };
}

double shape_area(const shape_t* shape) {
    // 패턴 매칭 스타일
    switch (shape->type) {
        case SHAPE_CIRCLE:
            return M_PI * shape->data.circle.radius * shape->data.circle.radius;
            
        case SHAPE_RECTANGLE:
            return shape->data.rectangle.width * shape->data.rectangle.height;
            
        case SHAPE_TRIANGLE:
            return 0.5 * shape->data.triangle.base * shape->data.triangle.height;
            
        default:
            return 0.0;
    }
}

void shape_print(const shape_t* shape) {
    switch (shape->type) {
        case SHAPE_CIRCLE:
            printf("Circle(radius=%.2f)", shape->data.circle.radius);
            break;
            
        case SHAPE_RECTANGLE:
            printf("Rectangle(%.2f x %.2f)", 
                   shape->data.rectangle.width, shape->data.rectangle.height);
            break;
            
        case SHAPE_TRIANGLE:
            printf("Triangle(base=%.2f, height=%.2f)", 
                   shape->data.triangle.base, shape->data.triangle.height);
            break;
    }
}
```

* usage
```c++
// test_shapes.c
#include "shape.h"
#include <stdio.h>

int main(void) {
    // 다양한 타입의 Shape 생성
    shape_t shapes[] = {
        shape_circle(5.0),
        shape_rectangle(4.0, 3.0),
        shape_triangle(6.0, 4.0)
    };
    
    // 동일한 인터페이스로 처리
    for (int i = 0; i < 3; i++) {
        shape_print(&shapes[i]);
        printf(" - Area: %.2f\n", shape_area(&shapes[i]));
    }
    
    return 0;
}
```

### 3. co-product with hiding implementation :: 권장

```c++
// shape.h
#ifndef SHAPE_H
#define SHAPE_H

// 불투명 포인터 선언
typedef struct shape shape_t;

// Shape 타입 enum (외부에 노출)
typedef enum {
    SHAPE_TYPE_CIRCLE,
    SHAPE_TYPE_RECTANGLE,
    SHAPE_TYPE_TRIANGLE
} shape_type_t;

// 팩토리 함수들
shape_t* shape_create_circle(double radius);
shape_t* shape_create_rectangle(double width, double height);
shape_t* shape_create_triangle(double base, double height);

// 소멸자
void shape_destroy(shape_t* shape);

// 인터페이스 함수들
shape_type_t shape_get_type(const shape_t* shape);
double shape_get_area(const shape_t* shape);
double shape_get_perimeter(const shape_t* shape);
void shape_print(const shape_t* shape);

// 방문자 패턴 지원
typedef struct {
    void (*visit_circle)(double radius, void* context);
    void (*visit_rectangle)(double width, double height, void* context);
    void (*visit_triangle)(double base, double height, void* context);
} shape_visitor_t;

void shape_accept(const shape_t* shape, const shape_visitor_t* visitor, void* context);

#endif
```

```c++
// shape.c
#include "shape.h"
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

// 내부 타입 태그 (외부에 숨김)
typedef enum {
    SHAPE_INTERNAL_CIRCLE,
    SHAPE_INTERNAL_RECTANGLE,
    SHAPE_INTERNAL_TRIANGLE
} shape_internal_type_t;

// 내부 데이터 구조체들
typedef struct {
    double radius;
} circle_data_t;

typedef struct {
    double width;
    double height;
} rectangle_data_t;

typedef struct {
    double base;
    double height;
} triangle_data_t;

// 실제 Shape 구조체 (완전히 숨김)
struct shape {
    shape_internal_type_t internal_type;
    union {
        circle_data_t circle;
        rectangle_data_t rectangle;
        triangle_data_t triangle;
    } data;
};

shape_t* shape_create_circle(double radius) {
    if (radius <= 0) return NULL;
    
    shape_t* shape = malloc(sizeof(struct shape));
    if (!shape) return NULL;
    
    shape->internal_type = SHAPE_INTERNAL_CIRCLE;
    shape->data.circle.radius = radius;
    return shape;
}

shape_t* shape_create_rectangle(double width, double height) {
    if (width <= 0 || height <= 0) return NULL;
    
    shape_t* shape = malloc(sizeof(struct shape));
    if (!shape) return NULL;
    
    shape->internal_type = SHAPE_INTERNAL_RECTANGLE;
    shape->data.rectangle.width = width;
    shape->data.rectangle.height = height;
    return shape;
}

shape_t* shape_create_triangle(double base, double height) {
    if (base <= 0 || height <= 0) return NULL;
    
    shape_t* shape = malloc(sizeof(struct shape));
    if (!shape) return NULL;
    
    shape->internal_type = SHAPE_INTERNAL_TRIANGLE;
    shape->data.triangle.base = base;
    shape->data.triangle.height = height;
    return shape;
}

void shape_destroy(shape_t* shape) {
    free(shape);  // 간단한 경우 - 복잡한 경우 내부 리소스도 정리
}

shape_type_t shape_get_type(const shape_t* shape) {
    if (!shape) return SHAPE_TYPE_CIRCLE;  // 기본값
    
    // 내부 타입을 외부 타입으로 변환
    switch (shape->internal_type) {
        case SHAPE_INTERNAL_CIRCLE: return SHAPE_TYPE_CIRCLE;
        case SHAPE_INTERNAL_RECTANGLE: return SHAPE_TYPE_RECTANGLE;
        case SHAPE_INTERNAL_TRIANGLE: return SHAPE_TYPE_TRIANGLE;
        default: return SHAPE_TYPE_CIRCLE;
    }
}

double shape_get_area(const shape_t* shape) {
    if (!shape) return 0.0;
    
    switch (shape->internal_type) {
        case SHAPE_INTERNAL_CIRCLE:
            return M_PI * shape->data.circle.radius * shape->data.circle.radius;
            
        case SHAPE_INTERNAL_RECTANGLE:
            return shape->data.rectangle.width * shape->data.rectangle.height;
            
        case SHAPE_INTERNAL_TRIANGLE:
            return 0.5 * shape->data.triangle.base * shape->data.triangle.height;
            
        default:
            return 0.0;
    }
}

void shape_print(const shape_t* shape) {
    if (!shape) return;
    
    switch (shape->internal_type) {
        case SHAPE_INTERNAL_CIRCLE:
            printf("Circle(radius=%.2f)", shape->data.circle.radius);
            break;
            
        case SHAPE_INTERNAL_RECTANGLE:
            printf("Rectangle(%.2f x %.2f)", 
                   shape->data.rectangle.width, shape->data.rectangle.height);
            break;
            
        case SHAPE_INTERNAL_TRIANGLE:
            printf("Triangle(base=%.2f, height=%.2f)", 
                   shape->data.triangle.base, shape->data.triangle.height);
            break;
    }
}

void shape_accept(const shape_t* shape, const shape_visitor_t* visitor, void* context) {
    if (!shape || !visitor) return;
    
    switch (shape->internal_type) {
        case SHAPE_INTERNAL_CIRCLE:
            if (visitor->visit_circle) {
                visitor->visit_circle(shape->data.circle.radius, context);
            }
            break;
            
        case SHAPE_INTERNAL_RECTANGLE:
            if (visitor->visit_rectangle) {
                visitor->visit_rectangle(shape->data.rectangle.width, 
                                       shape->data.rectangle.height, context);
            }
            break;
            
        case SHAPE_INTERNAL_TRIANGLE:
            if (visitor->visit_triangle) {
                visitor->visit_triangle(shape->data.triangle.base, 
                                      shape->data.triangle.height, context);
            }
            break;
    }
}
```


# Appendix

## 1. generic Optional :: 비권장

* Trick in modern library. 
* Don't make like this. (use library instead.)

```c++
// generic_optional.h
#ifndef GENERIC_OPTIONAL_H
#define GENERIC_OPTIONAL_H

// 매크로로 타입별 Optional 생성
#define DEFINE_OPTIONAL(T) \
    typedef enum { \
        OPTIONAL_##T##_SOME, \
        OPTIONAL_##T##_NONE \
    } optional_##T##_tag_t; \
    \
    typedef struct { \
        optional_##T##_tag_t tag; \
        union { \
            T value; \
        } data; \
    } optional_##T##_t; \
    \
    static inline optional_##T##_t optional_##T##_some(T value) { \
        return (optional_##T##_t) { \
            .tag = OPTIONAL_##T##_SOME, \
            .data.value = value \
        }; \
    } \
    \
    static inline optional_##T##_t optional_##T##_none(void) { \
        return (optional_##T##_t) { \
            .tag = OPTIONAL_##T##_NONE \
        }; \
    } \
    \
    static inline bool optional_##T##_is_some(const optional_##T##_t* opt) { \
        return opt && opt->tag == OPTIONAL_##T##_SOME; \
    }

// 사용법
DEFINE_OPTIONAL(int)     // optional_int_t 생성
DEFINE_OPTIONAL(double)  // optional_double_t 생성
DEFINE_OPTIONAL(char*)   // optional_char_ptr_t 생성

#endif
```

* usage

```c++
// test_generic.c
#include "generic_optional.h"
#include <stdio.h>

optional_int_t find_first_positive(int* arr, size_t len) {
    for (size_t i = 0; i < len; i++) {
        if (arr[i] > 0) {
            return optional_int_some(arr[i]);
        }
    }
    return optional_int_none();
}

int main(void) {
    int numbers[] = {-1, -2, 3, 4, -5};
    
    optional_int_t result = find_first_positive(numbers, 5);
    
    if (optional_int_is_some(&result)) {
        printf("First positive: %d\n", result.data.value);
    } else {
        printf("No positive number found\n");
    }
    
    return 0;
}
```

### 2. 다형성 흉내내기 :: 비권장

* 아래의 샘플은 **매우 간단한** 사례임.
* 향후에도 추가/삭제/변경될 가능성이 극히 적은 공통 interface를 정의하는 것 자체가 매우 어려움
* 라이브러리를 만드는 상황이 아니라면, 피해야 하는 기법.
* 다형성 기법을 심각하게 써야 한다면, C++을 써라.


```c++
// advanced_file.h
#ifndef ADVANCED_FILE_H
#define ADVANCED_FILE_H

typedef struct file_manager file_manager_t;

typedef enum {
    FILE_TYPE_TEXT,
    FILE_TYPE_BINARY,
    FILE_TYPE_COMPRESSED
} file_type_t;

// Factory 패턴 + 불투명 포인터
file_manager_t* file_manager_create(file_type_t type);
void file_manager_destroy(file_manager_t* manager);
bool file_manager_open(file_manager_t* manager, const char* filename);
bool file_manager_save(file_manager_t* manager, const char* filename);
size_t file_manager_read_line(file_manager_t* manager, char* buffer, size_t size);

#endif
```

```c++
// advanced_file.c
#include "advanced_file.h"
#include <stdlib.h>

// 내부 구조체에 함수 포인터로 다형성 구현
struct file_manager {
    file_type_t type;
    void* internal_handle;
    
    // 가상 함수들
    bool (*open_func)(struct file_manager* self, const char* filename);
    bool (*save_func)(struct file_manager* self, const char* filename);
    size_t (*read_line_func)(struct file_manager* self, char* buffer, size_t size);
    void (*cleanup_func)(struct file_manager* self);
};

// 내부 함수들 (static으로 숨김)
static bool text_file_open(struct file_manager* self, const char* filename) {
    // 텍스트 파일 열기 구현
    return true;
}

static bool binary_file_open(struct file_manager* self, const char* filename) {
    // 바이너리 파일 열기 구현  
    return true;
}

// 생성자
// --------------------------------------------------------------------------------
file_manager_t* file_manager_create(file_type_t type) {
    file_manager_t* manager = malloc(sizeof(file_manager_t));
    if (!manager) return NULL;
    
    manager->type = type;
    manager->internal_handle = NULL;
    
    // 타입에 따라 다른 함수 포인터 할당
    switch (type) {
        case FILE_TYPE_TEXT:
            manager->open_func = text_file_open;
            // ... 다른 함수들
            break;
        case FILE_TYPE_BINARY:
            manager->open_func = binary_file_open;
            // ... 다른 함수들
            break;
        default:
            free(manager);
            return NULL;
    }
    
    return manager;
}

bool file_manager_open(file_manager_t* manager, const char* filename) {
    if (!manager || !manager->open_func) return false;
    
    // 다형성 - 타입에 따라 다른 함수 호출
    return manager->open_func(manager, filename);
}
```
