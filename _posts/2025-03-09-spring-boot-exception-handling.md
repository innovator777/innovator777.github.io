---
layout: post
title: 스프링 부트 Exception Handling 전략 완벽 가이드
date: 2025-03-09
categories: [Spring Boot]
tags: [spring, spring boot, exception handling, 예외 처리, global exception handler, 오류 처리]
---

이전 포스팅에서는 HTTP 메서드별 처리 방법에 대해 알아보았습니다. 이번에는 스프링 부트 애플리케이션에서 예외 처리(Exception Handling)를 효과적으로 구현하는 방법에 대해 자세히 알아보겠습니다. 적절한 예외 처리는 애플리케이션의 안정성과 사용자 경험을 크게 향상시키는 중요한 요소입니다.

## 예외 처리의 중요성

예외 처리는 다음과 같은 이유로 애플리케이션 개발에서 매우 중요합니다:

1. **사용자 경험 향상**: 사용자에게 이해하기 쉬운 오류 메시지 제공
2. **보안 강화**: 민감한 오류 정보가 외부로 노출되는 것을 방지
3. **디버깅 용이성**: 개발자가 문제를 빠르게 파악하고 해결할 수 있도록 지원
4. **애플리케이션 안정성**: 예외 상황에서도 애플리케이션이 중단되지 않고 계속 실행되도록 보장

## 스프링 부트의 예외 처리 방식

스프링 부트는 다양한 예외 처리 메커니즘을 제공합니다. 주요 방식은 다음과 같습니다:

### 1. @ExceptionHandler

`@ExceptionHandler` 어노테이션은 컨트롤러 내에서 발생하는 특정 예외를 처리하는 메서드를 지정합니다.

```java
@RestController
public class ProductController {
    
    // 일반 API 엔드포인트들...
    
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleProductNotFoundException(ProductNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("PRODUCT_NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

이 방식의 단점은 해당 컨트롤러 내에서만 예외 처리가 적용된다는 것입니다. 여러 컨트롤러에서 동일한 예외 처리 로직이 필요한 경우 중복 코드가 발생할 수 있습니다.

### 2. @ControllerAdvice와 @RestControllerAdvice

`@ControllerAdvice`와 `@RestControllerAdvice`는 전역적인 예외 처리를 구현할 수 있게 해주는 어노테이션입니다. 이를 통해 애플리케이션 전체에서 발생하는 예외를 일관되게 처리할 수 있습니다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleProductNotFoundException(ProductNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("PRODUCT_NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(InvalidRequestException.class)
    public ResponseEntity<ErrorResponse> handleInvalidRequestException(InvalidRequestException ex) {
        ErrorResponse error = new ErrorResponse("INVALID_REQUEST", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse("INTERNAL_SERVER_ERROR", "An unexpected error occurred");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

`@RestControllerAdvice`는 `@ControllerAdvice`와 `@ResponseBody`를 결합한 어노테이션으로, REST API에서 예외 처리 시 주로 사용됩니다.

### 3. ResponseStatusException

Spring 5부터 도입된 `ResponseStatusException`은 예외 클래스를 별도로 생성하지 않고도 HTTP 상태 코드와 오류 메시지를 지정할 수 있는 방법을 제공합니다.

```java
@GetMapping("/products/{id}")
public Product getProduct(@PathVariable Long id) {
    return productRepository.findById(id)
        .orElseThrow(() -> new ResponseStatusException(
            HttpStatus.NOT_FOUND, "Product not found with id: " + id));
}
```

이 방식은 간단한 예외 처리에 적합하며, 코드를 간결하게 유지할 수 있습니다.

## 커스텀 예외 클래스 설계

효과적인 예외 처리를 위해 애플리케이션 도메인에 맞는 커스텀 예외 클래스를 설계하는 것이 좋습니다.

```java
// 기본 예외 클래스
public abstract class BaseException extends RuntimeException {
    private final String errorCode;
    
    public BaseException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// 리소스를 찾을 수 없을 때 사용하는 예외
public class ResourceNotFoundException extends BaseException {
    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super("RESOURCE_NOT_FOUND", 
              String.format("%s not found with %s: '%s'", resourceName, fieldName, fieldValue));
    }
}

// 유효성 검사 실패 시 사용하는 예외
public class ValidationException extends BaseException {
    private final Map<String, String> errors;
    
    public ValidationException(Map<String, String> errors) {
        super("VALIDATION_FAILED", "Validation failed");
        this.errors = errors;
    }
    
    public Map<String, String> getErrors() {
        return errors;
    }
}
```

## 표준화된 오류 응답 형식

클라이언트에게 일관된 오류 응답을 제공하기 위해 표준화된 오류 응답 형식을 정의하는 것이 중요합니다.

```java
public class ErrorResponse {
    private final String timestamp;
    private final String errorCode;
    private final String message;
    private Map<String, String> errors;
    
    public ErrorResponse(String errorCode, String message) {
        this.timestamp = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSXXX")
            .format(new Date());
        this.errorCode = errorCode;
        this.message = message;
    }
    
    // 유효성 검사 오류와 같은 세부 오류 정보를 포함할 때 사용
    public ErrorResponse(String errorCode, String message, Map<String, String> errors) {
        this(errorCode, message);
        this.errors = errors;
    }
    
    // Getters
    public String getTimestamp() {
        return timestamp;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
    
    public String getMessage() {
        return message;
    }
    
    public Map<String, String> getErrors() {
        return errors;
    }
}
```

## 전역 예외 처리기 구현 예제

다음은 다양한 예외를 처리하는 전역 예외 처리기의 전체 구현 예제입니다:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    // 리소스를 찾을 수 없는 경우
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(ex.getErrorCode(), ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    // 요청 본문 파싱 오류
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleHttpMessageNotReadableException(HttpMessageNotReadableException ex) {
        ErrorResponse error = new ErrorResponse("BAD_REQUEST", "Malformed JSON request");
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    // 메서드 인자 유효성 검사 실패
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage()));
        
        ErrorResponse errorResponse = new ErrorResponse("VALIDATION_FAILED", "Validation failed", errors);
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    // 커스텀 유효성 검사 예외
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        ErrorResponse errorResponse = new ErrorResponse(ex.getErrorCode(), ex.getMessage(), ex.getErrors());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
    
    // 접근 권한 없음
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDeniedException(AccessDeniedException ex) {
        ErrorResponse error = new ErrorResponse("ACCESS_DENIED", "Access denied");
        return new ResponseEntity<>(error, HttpStatus.FORBIDDEN);
    }
    
    // 기타 모든 예외
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        logger.error("Unhandled exception occurred", ex);
        ErrorResponse error = new ErrorResponse("INTERNAL_SERVER_ERROR", "An unexpected error occurred");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## 예외 처리 모범 사례

### 1. 계층별 예외 처리

애플리케이션의 각 계층(컨트롤러, 서비스, 리포지토리 등)에 맞는 예외 처리 전략을 수립하세요.

```java
// 서비스 계층
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public Product getProductById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", "id", id));
    }
}

// 컨트롤러 계층
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }
}
```

### 2. 로깅 전략

예외 발생 시 적절한 로깅을 통해 문제 해결에 필요한 정보를 기록하세요.

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleGenericException(Exception ex, WebRequest request) {
    // 요청 정보와 함께 상세 로그 기록
    logger.error("Unhandled exception occurred. Request: {}", request.getDescription(false), ex);
    
    // 클라이언트에게는 민감한 정보를 제외한 일반적인 오류 메시지 반환
    ErrorResponse error = new ErrorResponse("INTERNAL_SERVER_ERROR", "An unexpected error occurred");
    return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
}
```

### 3. 환경별 오류 응답 차별화

개발 환경과 운영 환경에서 오류 응답의 상세 수준을 다르게 설정하세요.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @Value("${spring.profiles.active:prod}")
    private String activeProfile;
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        logger.error("Unhandled exception occurred", ex);
        
        ErrorResponse error;
        if ("dev".equals(activeProfile) || "test".equals(activeProfile)) {
            // 개발/테스트 환경: 상세 오류 정보 포함
            error = new ErrorResponse("INTERNAL_SERVER_ERROR", ex.getMessage());
            // 스택 트레이스 등 추가 정보를 포함할 수도 있음
        } else {
            // 운영 환경: 일반적인 오류 메시지만 포함
            error = new ErrorResponse("INTERNAL_SERVER_ERROR", "An unexpected error occurred");
        }
        
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## 결론

효과적인 예외 처리는 견고하고 사용자 친화적인 애플리케이션을 구축하는 데 필수적입니다. 스프링 부트는 `@ExceptionHandler`, `@ControllerAdvice`, `ResponseStatusException` 등 다양한 예외 처리 메커니즘을 제공하여 개발자가 상황에 맞는 예외 처리 전략을 구현할 수 있도록 지원합니다.

표준화된 오류 응답 형식을 정의하고, 애플리케이션 도메인에 맞는 커스텀 예외 클래스를 설계하며, 전역 예외 처리기를 구현하여 일관된 예외 처리를 구현하세요. 또한, 적절한 로깅과 환경별 오류 응답 차별화를 통해 개발 및 운영 과정에서의 문제 해결을 용이하게 할 수 있습니다.

다음 포스팅에서는 스프링 부트에서 데이터 유효성 검사(Validation)를 구현하는 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Boot 공식 문서 - 오류 처리](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.error-handling)
- [Spring Framework 공식 문서 - 예외 처리](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)
- [Baeldung - Spring REST 오류 처리](https://www.baeldung.com/exception-handling-for-rest-with-spring)
- [Spring Boot in Action](https://www.manning.com/books/spring-boot-in-action)
- [RESTful API 설계 모범 사례 - 오류 처리](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#7102-error-condition-responses) 