---
layout: post
title: 스프링 부트에서 HTTP 메서드별 처리방법 완벽 가이드
date: 2025-03-08
categories: [Spring Boot]
tags: [spring, spring boot, http methods, rest api, get, post, put, delete, patch]
---

이전 포스팅에서는 `@Controller`와 `@RestController`의 차이점에 대해 알아보았습니다. 이번에는 RESTful API 개발에서 핵심이 되는 HTTP 메서드(GET, POST, PUT, DELETE, PATCH)별 처리 방법에 대해 자세히 알아보겠습니다. 각 HTTP 메서드의 특징과 스프링 부트에서 이를 처리하는 방법을 코드 예제와 함께 살펴보겠습니다.

## HTTP 메서드의 이해

HTTP 메서드는 클라이언트가 서버에 요청하는 작업의 종류를 나타냅니다. RESTful API에서는 이러한 HTTP 메서드를 사용하여 CRUD(Create, Read, Update, Delete) 작업을 표현합니다.

| HTTP 메서드 | CRUD 작업 | 설명 |
|------------|-----------|------|
| GET | Read | 리소스 조회 |
| POST | Create | 리소스 생성 |
| PUT | Update/Replace | 리소스 전체 교체 |
| PATCH | Update/Modify | 리소스 부분 수정 |
| DELETE | Delete | 리소스 삭제 |

## 스프링 부트에서의 HTTP 메서드 처리

스프링 부트에서는 `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` 어노테이션을 사용하여 각 HTTP 메서드에 대한 핸들러 메서드를 정의할 수 있습니다.

### 1. GET 메서드 처리

GET 메서드는 서버로부터 데이터를 조회할 때 사용합니다. 스프링 부트에서는 `@GetMapping` 어노테이션을 사용하여 GET 요청을 처리합니다.

#### 모든 리소스 조회

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }
}
```

#### 특정 리소스 조회

```java
@GetMapping("/{id}")
public ResponseEntity<Product> getProductById(@PathVariable Long id) {
    return productService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

#### 쿼리 파라미터를 사용한 필터링

```java
@GetMapping
public List<Product> getProducts(
        @RequestParam(required = false) String category,
        @RequestParam(required = false) Double minPrice,
        @RequestParam(required = false) Double maxPrice) {
    
    if (category != null && minPrice != null && maxPrice != null) {
        return productService.findByCategoryAndPriceRange(category, minPrice, maxPrice);
    } else if (category != null) {
        return productService.findByCategory(category);
    } else if (minPrice != null && maxPrice != null) {
        return productService.findByPriceRange(minPrice, maxPrice);
    } else {
        return productService.findAll();
    }
}
```

### 2. POST 메서드 처리

POST 메서드는 새로운 리소스를 생성할 때 사용합니다. 스프링 부트에서는 `@PostMapping` 어노테이션을 사용하여 POST 요청을 처리합니다.

```java
@PostMapping
public ResponseEntity<Product> createProduct(@RequestBody @Valid ProductDto productDto) {
    Product savedProduct = productService.save(productDto.toEntity());
    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(savedProduct.getId())
            .toUri();
    
    return ResponseEntity.created(location).body(savedProduct);
}
```

위 예제에서는 다음과 같은 특징이 있습니다:
- `@RequestBody`: HTTP 요청 본문을 자바 객체로 변환
- `@Valid`: 요청 데이터의 유효성 검사
- `ResponseEntity.created()`: 201 Created 상태 코드와 함께 생성된 리소스의 위치를 헤더에 포함

### 3. PUT 메서드 처리

PUT 메서드는 리소스를 완전히 교체할 때 사용합니다. 스프링 부트에서는 `@PutMapping` 어노테이션을 사용하여 PUT 요청을 처리합니다.

```java
@PutMapping("/{id}")
public ResponseEntity<Product> updateProduct(
        @PathVariable Long id,
        @RequestBody @Valid ProductDto productDto) {
    
    if (!productService.existsById(id)) {
        return ResponseEntity.notFound().build();
    }
    
    Product product = productDto.toEntity();
    product.setId(id);
    Product updatedProduct = productService.save(product);
    
    return ResponseEntity.ok(updatedProduct);
}
```

PUT 메서드의 특징:
- 리소스의 모든 필드를 업데이트
- 요청에 포함되지 않은 필드는 기본값 또는 null로 설정
- 리소스가 존재하지 않으면 404 Not Found 반환

### 4. PATCH 메서드 처리

PATCH 메서드는 리소스의 일부만 수정할 때 사용합니다. 스프링 부트에서는 `@PatchMapping` 어노테이션을 사용하여 PATCH 요청을 처리합니다.

```java
@PatchMapping("/{id}")
public ResponseEntity<Product> partialUpdateProduct(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates) {
    
    return productService.findById(id)
        .map(existingProduct -> {
            // 제공된 필드만 업데이트
            if (updates.containsKey("name")) {
                existingProduct.setName((String) updates.get("name"));
            }
            if (updates.containsKey("price")) {
                existingProduct.setPrice(Double.valueOf(updates.get("price").toString()));
            }
            if (updates.containsKey("category")) {
                existingProduct.setCategory((String) updates.get("category"));
            }
            
            Product updatedProduct = productService.save(existingProduct);
            return ResponseEntity.ok(updatedProduct);
        })
        .orElse(ResponseEntity.notFound().build());
}
```

PATCH 메서드의 특징:
- 리소스의 일부 필드만 업데이트
- 요청에 포함된 필드만 변경
- 리소스가 존재하지 않으면 404 Not Found 반환

### 5. DELETE 메서드 처리

DELETE 메서드는 리소스를 삭제할 때 사용합니다. 스프링 부트에서는 `@DeleteMapping` 어노테이션을 사용하여 DELETE 요청을 처리합니다.

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
    return productService.findById(id)
        .map(product -> {
            productService.deleteById(id);
            return ResponseEntity.noContent().<Void>build();
        })
        .orElse(ResponseEntity.notFound().build());
}
```

DELETE 메서드의 특징:
- 성공적으로 삭제되면 204 No Content 상태 코드 반환
- 리소스가 존재하지 않으면 404 Not Found 반환

## HTTP 메서드별 응답 상태 코드

각 HTTP 메서드에 대한 일반적인 응답 상태 코드는 다음과 같습니다:

| HTTP 메서드 | 성공 시 상태 코드 | 실패 시 상태 코드 |
|------------|-----------------|-----------------|
| GET | 200 OK | 404 Not Found |
| POST | 201 Created | 400 Bad Request |
| PUT | 200 OK | 400 Bad Request, 404 Not Found |
| PATCH | 200 OK | 400 Bad Request, 404 Not Found |
| DELETE | 204 No Content | 404 Not Found |

## 실제 애플리케이션 예제: 사용자 관리 API

다음은 사용자 관리 API의 전체 컨트롤러 예제입니다:

```java
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
    public ResponseEntity<User> createUser(@RequestBody @Valid UserDto userDto) {
        User savedUser = userService.save(userDto.toEntity());
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(savedUser.getId())
                .toUri();
        
        return ResponseEntity.created(location).body(savedUser);
    }
    
    // 사용자 전체 정보 업데이트
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id,
            @RequestBody @Valid UserDto userDto) {
        
        if (!userService.existsById(id)) {
            return ResponseEntity.notFound().build();
        }
        
        User user = userDto.toEntity();
        user.setId(id);
        User updatedUser = userService.save(user);
        
        return ResponseEntity.ok(updatedUser);
    }
    
    // 사용자 부분 정보 업데이트
    @PatchMapping("/{id}")
    public ResponseEntity<User> partialUpdateUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        
        return userService.findById(id)
            .map(existingUser -> {
                if (updates.containsKey("name")) {
                    existingUser.setName((String) updates.get("name"));
                }
                if (updates.containsKey("email")) {
                    existingUser.setEmail((String) updates.get("email"));
                }
                if (updates.containsKey("role")) {
                    existingUser.setRole((String) updates.get("role"));
                }
                
                User updatedUser = userService.save(existingUser);
                return ResponseEntity.ok(updatedUser);
            })
            .orElse(ResponseEntity.notFound().build());
    }
    
    // 사용자 삭제
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(user -> {
                userService.deleteById(id);
                return ResponseEntity.noContent().<Void>build();
            })
            .orElse(ResponseEntity.notFound().build());
    }
}
```

## HTTP 메서드 처리 시 주의사항

### 1. 멱등성(Idempotency)

멱등성이란 동일한 요청을 여러 번 수행해도 결과가 동일함을 의미합니다.

- GET, PUT, DELETE는 멱등성을 가짐
- POST는 멱등성을 가지지 않음 (여러 번 호출하면 여러 개의 리소스가 생성됨)
- PATCH는 구현에 따라 멱등성을 가질 수도, 가지지 않을 수도 있음

### 2. 안전성(Safety)

안전한 메서드는 리소스의 상태를 변경하지 않는 메서드입니다.

- GET은 안전한 메서드
- POST, PUT, PATCH, DELETE는 안전하지 않은 메서드

### 3. 캐시 가능성(Cacheability)

- GET 요청은 캐시 가능
- POST, PUT, DELETE, PATCH는 일반적으로 캐시되지 않음

## 결론

스프링 부트에서는 `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping` 어노테이션을 사용하여 각 HTTP 메서드에 대한 핸들러를 쉽게 구현할 수 있습니다. 각 HTTP 메서드의 특성과 용도를 이해하고 적절히 활용하면, RESTful API를 설계하고 구현하는 데 큰 도움이 됩니다.

HTTP 메서드의 특성(멱등성, 안전성, 캐시 가능성)을 고려하여 API를 설계하고, 적절한 상태 코드를 반환하는 것이 중요합니다. 이를 통해 클라이언트가 API의 동작을 쉽게 이해하고 예측할 수 있는 일관된 인터페이스를 제공할 수 있습니다.

다음 포스팅에서는 스프링 부트에서 데이터 유효성 검사(Validation)를 구현하는 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - 요청 매핑](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping)
- [HTTP 메서드 명세 - RFC 7231](https://tools.ietf.org/html/rfc7231#section-4)
- [RESTful API 설계 가이드](https://restfulapi.net/http-methods/)
- [Baeldung - HTTP PUT vs HTTP PATCH](https://www.baeldung.com/http-put-patch-difference-spring)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 