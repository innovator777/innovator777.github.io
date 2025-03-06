---
layout: post
title: 스프링 부트로 REST API 개발하기 - 기초편
date: 2025-03-06
categories: [Spring Boot]
tags: [spring, spring boot, rest api, restful, web, 레스트 API]
---

이전 포스팅에서는 스프링 부트의 프로파일 설정과 활용에 대해 알아보았습니다. 이번에는 스프링 부트를 사용하여 REST API를 개발하는 기초적인 방법에 대해 알아보겠습니다. REST API는 현대 웹 애플리케이션과 모바일 앱 개발에서 필수적인 요소로, 스프링 부트는 이를 쉽게 구현할 수 있는 강력한 도구를 제공합니다.

## REST API란?

REST(Representational State Transfer)는 웹 서비스를 설계하고 구현하기 위한 아키텍처 스타일입니다. REST API는 다음과 같은 특징을 가집니다:

1. **자원(Resource) 기반**: 모든 것을 자원으로 표현하고, 각 자원은 고유한 URI를 가짐
2. **HTTP 메서드 활용**: GET, POST, PUT, DELETE 등의 HTTP 메서드를 사용하여 자원에 대한 CRUD 작업 수행
3. **상태 없음(Stateless)**: 각 요청은 독립적이며, 서버는 클라이언트의 상태를 저장하지 않음
4. **표현(Representation)**: 자원은 JSON, XML 등 다양한 형식으로 표현될 수 있음

## 스프링 부트에서 REST API 개발 준비

### 1. 의존성 추가

REST API 개발을 위해 필요한 기본 의존성은 `spring-boot-starter-web`입니다.

**Maven**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**Gradle**:
```gradle
implementation 'org.springframework.boot:spring-boot-starter-web'
```

### 2. 기본 프로젝트 구조

REST API 개발을 위한 기본적인 프로젝트 구조는 다음과 같습니다:

```
src/main/java/com/example/demo/
├── DemoApplication.java
├── controller/
│   └── UserController.java
├── model/
│   └── User.java
├── repository/
│   └── UserRepository.java
├── service/
│   └── UserService.java
└── exception/
    └── ResourceNotFoundException.java
```

## REST 컨트롤러 구현하기

### 1. @RestController 어노테이션

스프링 부트에서는 `@RestController` 어노테이션을 사용하여 REST API 엔드포인트를 쉽게 구현할 수 있습니다.

```java
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
@RequestMapping("/api")
public class UserController {
    // API 엔드포인트 메서드들
}
```

`@RestController`는 `@Controller`와 `@ResponseBody`를 결합한 어노테이션으로, 모든 메서드의 반환값이 HTTP 응답 본문에 직접 매핑됩니다.

### 2. 기본 CRUD 엔드포인트 구현

#### 모델 클래스 정의

```java
public class User {
    private Long id;
    private String name;
    private String email;
    
    // 생성자, getter, setter
}
```

#### 서비스 계층 구현

```java
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class UserService {
    private final ConcurrentHashMap<Long, User> users = new ConcurrentHashMap<>();
    private final AtomicLong counter = new AtomicLong();
    
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
    
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(users.get(id));
    }
    
    public User save(User user) {
        if (user.getId() == null) {
            user.setId(counter.incrementAndGet());
        }
        users.put(user.getId(), user);
        return user;
    }
    
    public boolean deleteById(Long id) {
        return users.remove(id) != null;
    }
}
```

#### 컨트롤러 구현

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // 모든 사용자 조회
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    // 특정 사용자 조회
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    // 사용자 생성
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
    
    // 사용자 수정
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.findById(id)
                .map(existingUser -> {
                    user.setId(id);
                    return ResponseEntity.ok(userService.save(user));
                })
                .orElse(ResponseEntity.notFound().build());
    }
    
    // 사용자 삭제
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        return userService.deleteById(id)
                ? ResponseEntity.noContent().build()
                : ResponseEntity.notFound().build();
    }
}
```

## HTTP 상태 코드 활용

RESTful API에서는 적절한 HTTP 상태 코드를 반환하는 것이 중요합니다:

- **200 OK**: 요청 성공
- **201 Created**: 리소스 생성 성공
- **204 No Content**: 요청 성공했지만 반환할 내용 없음 (예: 삭제 성공)
- **400 Bad Request**: 잘못된 요청
- **404 Not Found**: 리소스를 찾을 수 없음
- **500 Internal Server Error**: 서버 오류

스프링 부트에서는 `ResponseEntity` 클래스를 사용하여 상태 코드와 응답 본문을 함께 반환할 수 있습니다:

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUserById(@PathVariable Long id) {
    return userService.findById(id)
            .map(user -> ResponseEntity.ok(user))  // 200 OK
            .orElse(ResponseEntity.notFound().build());  // 404 Not Found
}
```

## 요청 파라미터 처리

### 1. 경로 변수 (@PathVariable)

URL 경로에 포함된 변수를 추출할 때 사용합니다:

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUserById(@PathVariable Long id) {
    // id 값을 사용하여 사용자 조회
}
```

### 2. 쿼리 파라미터 (@RequestParam)

URL의 쿼리 문자열에서 파라미터를 추출할 때 사용합니다:

```java
@GetMapping
public List<User> searchUsers(
        @RequestParam(required = false) String name,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    // 이름으로 검색하고 페이징 처리
}
```

### 3. 요청 본문 (@RequestBody)

HTTP 요청 본문을 자바 객체로 변환할 때 사용합니다:

```java
@PostMapping
public User createUser(@RequestBody User user) {
    // 요청 본문의 JSON을 User 객체로 변환
    return userService.save(user);
}
```

## 예외 처리

REST API에서는 적절한 예외 처리가 중요합니다. 스프링 부트에서는 `@ControllerAdvice`와 `@ExceptionHandler`를 사용하여 전역 예외 처리를 구현할 수 있습니다:

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_PROVIDED);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex) {
        ErrorResponse error = new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
    
    // ErrorResponse 내부 클래스
    public static class ErrorResponse {
        private String code;
        private String message;
        
        public ErrorResponse(String code, String message) {
            this.code = code;
            this.message = message;
        }
        
        // getter, setter
    }
}
```

## 데이터 검증

클라이언트로부터 받은 데이터를 검증하기 위해 Bean Validation API를 사용할 수 있습니다:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

모델 클래스에 검증 어노테이션을 추가합니다:

```java
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

public class User {
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    // 생성자, getter, setter
}
```

컨트롤러에서 `@Valid` 어노테이션을 사용하여 검증을 활성화합니다:

```java
@PostMapping
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    User savedUser = userService.save(user);
    return new ResponseEntity<>(savedUser, HttpStatus.CREATED);
}
```

검증 실패 시 예외 처리:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors().forEach(error -> 
        errors.put(error.getField(), error.getDefaultMessage())
    );
    
    ErrorResponse errorResponse = new ErrorResponse("VALIDATION_FAILED", "Validation failed", errors);
    return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
}
```

## API 문서화

Swagger/OpenAPI를 사용하여 API 문서를 자동으로 생성할 수 있습니다:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.9</version>
</dependency>
```

기본 설정:

```java
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("User Management API")
                        .version("1.0")
                        .description("API for managing users"));
    }
}
```

애플리케이션을 실행한 후 `http://localhost:8080/swagger-ui.html`에서 API 문서를 확인할 수 있습니다.

## 실제 예제: 할 일 관리 API

간단한 할 일 관리 API를 구현해 보겠습니다:

### Todo 모델

```java
import java.time.LocalDateTime;

public class Todo {
    private Long id;
    private String title;
    private String description;
    private boolean completed;
    private LocalDateTime createdAt;
    
    // 생성자, getter, setter
}
```

### TodoService

```java
@Service
public class TodoService {
    private final Map<Long, Todo> todos = new ConcurrentHashMap<>();
    private final AtomicLong counter = new AtomicLong();
    
    public List<Todo> findAll() {
        return new ArrayList<>(todos.values());
    }
    
    public Optional<Todo> findById(Long id) {
        return Optional.ofNullable(todos.get(id));
    }
    
    public Todo save(Todo todo) {
        if (todo.getId() == null) {
            todo.setId(counter.incrementAndGet());
            todo.setCreatedAt(LocalDateTime.now());
        }
        todos.put(todo.getId(), todo);
        return todo;
    }
    
    public boolean deleteById(Long id) {
        return todos.remove(id) != null;
    }
    
    public List<Todo> findByCompleted(boolean completed) {
        return todos.values().stream()
                .filter(todo -> todo.isCompleted() == completed)
                .collect(Collectors.toList());
    }
}
```

### TodoController

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    
    private final TodoService todoService;
    
    @Autowired
    public TodoController(TodoService todoService) {
        this.todoService = todoService;
    }
    
    @GetMapping
    public List<Todo> getAllTodos(@RequestParam(required = false) Boolean completed) {
        if (completed != null) {
            return todoService.findByCompleted(completed);
        }
        return todoService.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Todo> getTodoById(@PathVariable Long id) {
        return todoService.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<Todo> createTodo(@Valid @RequestBody Todo todo) {
        Todo savedTodo = todoService.save(todo);
        return new ResponseEntity<>(savedTodo, HttpStatus.CREATED);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Todo> updateTodo(@PathVariable Long id, @Valid @RequestBody Todo todo) {
        return todoService.findById(id)
                .map(existingTodo -> {
                    todo.setId(id);
                    todo.setCreatedAt(existingTodo.getCreatedAt());
                    return ResponseEntity.ok(todoService.save(todo));
                })
                .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id) {
        return todoService.deleteById(id)
                ? ResponseEntity.noContent().build()
                : ResponseEntity.notFound().build();
    }
}
```

## 결론

스프링 부트는 REST API 개발을 위한 강력하고 유연한 도구를 제공합니다. `@RestController`, `@RequestMapping`, `@GetMapping`, `@PostMapping` 등의 어노테이션을 사용하여 간결하고 직관적인 API 엔드포인트를 구현할 수 있습니다.

이번 포스팅에서는 REST API의 기본 개념과 스프링 부트를 사용한 구현 방법에 대해 알아보았습니다. 적절한 HTTP 메서드와 상태 코드를 사용하고, 요청 파라미터를 처리하며, 예외를 적절히 처리하는 방법을 배웠습니다.

다음 포스팅에서는 스프링 데이터 JPA를 사용하여 데이터베이스와 연동하는 REST API 개발 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring REST 튜토리얼](https://spring.io/guides/tutorials/rest/)
- [RESTful API 설계 가이드](https://restfulapi.net/)
- [Baeldung - REST with Spring](https://www.baeldung.com/rest-with-spring-series)
- [Spring Boot in Action](https://www.manning.com/books/spring-boot-in-action) 