---
layout: post
title: 스프링의 @Controller와 @RestController 차이 이해하기
date: 2025-03-07
categories: [Spring Boot]
tags: [spring, spring boot, controller, restcontroller, mvc, rest api]
---

이전 포스팅에서는 스프링 부트로 REST API를 개발하는 기초적인 방법에 대해 알아보았습니다. 이번에는 스프링 MVC의 핵심 어노테이션인 `@Controller`와 `@RestController`의 차이점에 대해 자세히 알아보겠습니다. 이 두 어노테이션은 웹 애플리케이션 개발에서 자주 사용되지만, 각각의 목적과 동작 방식에는 중요한 차이가 있습니다.

## @Controller와 @RestController의 기본 개념

### @Controller

`@Controller`는 스프링 MVC에서 전통적인 웹 애플리케이션의 컨트롤러를 정의하는 데 사용됩니다. 주로 HTML 뷰를 반환하는 웹 페이지를 제공하는 데 초점을 맞추고 있습니다.

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
    
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Welcome to Spring MVC!");
        return "home"; // home.html 뷰를 반환
    }
}
```

### @RestController

`@RestController`는 RESTful 웹 서비스를 제공하기 위한 특수한 컨트롤러로, `@Controller`와 `@ResponseBody`를 결합한 어노테이션입니다. 주로 JSON/XML 형태의 데이터를 반환하는 API 엔드포인트를 구현하는 데 사용됩니다.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserApiController {
    
    @GetMapping("/api/users")
    public List<User> getUsers() {
        // 사용자 목록 반환
        return userService.findAll(); // 객체가 JSON으로 자동 변환됨
    }
}
```

## 주요 차이점

### 1. 반환 값 처리

**@Controller**:
- 기본적으로 뷰 이름(String)을 반환
- 반환된 문자열은 ViewResolver에 의해 처리되어 해당 이름의 뷰 템플릿을 찾아 렌더링
- 데이터는 Model 객체를 통해 뷰에 전달

**@RestController**:
- 객체를 직접 반환하면 자동으로 JSON/XML로 변환 (기본값은 JSON)
- 반환 값이 HTTP 응답 본문(Response Body)에 직접 작성됨
- 별도의 뷰 처리 과정이 없음

### 2. @ResponseBody 사용

**@Controller**:
- 데이터를 응답 본문에 직접 작성하려면 메서드에 `@ResponseBody` 어노테이션을 추가해야 함

```java
@Controller
public class DataController {
    
    @GetMapping("/data")
    @ResponseBody
    public User getData() {
        return new User("John", "john@example.com");
    }
}
```

**@RestController**:
- 클래스 수준에서 이미 `@ResponseBody`가 포함되어 있어 모든 메서드에 자동 적용
- 추가로 `@ResponseBody`를 명시할 필요 없음

### 3. 사용 사례

**@Controller**:
- 전통적인 웹 애플리케이션 (서버 사이드 렌더링)
- Thymeleaf, JSP 등의 템플릿 엔진을 사용한 뷰 반환
- 폼 제출 처리 및 웹 페이지 네비게이션

**@RestController**:
- RESTful 웹 서비스 개발
- 싱글 페이지 애플리케이션(SPA)의 백엔드 API
- 모바일 애플리케이션을 위한 API 엔드포인트
- 마이크로서비스 간 통신

## 코드로 보는 차이점

### @Controller 예제 (뷰 반환)

```java
@Controller
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/products")
    public String listProducts(Model model) {
        List<Product> products = productService.findAll();
        model.addAttribute("products", products);
        return "product/list"; // product/list.html 뷰 템플릿 반환
    }
    
    @GetMapping("/products/{id}")
    public String productDetail(@PathVariable Long id, Model model) {
        Product product = productService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
        model.addAttribute("product", product);
        return "product/detail"; // product/detail.html 뷰 템플릿 반환
    }
    
    @PostMapping("/products")
    public String createProduct(@ModelAttribute @Valid ProductForm form, 
                               BindingResult result, 
                               RedirectAttributes attributes) {
        if (result.hasErrors()) {
            return "product/form"; // 유효성 검사 실패 시 폼 페이지로 돌아감
        }
        
        Product product = new Product();
        product.setName(form.getName());
        product.setPrice(form.getPrice());
        productService.save(product);
        
        attributes.addFlashAttribute("message", "Product created successfully!");
        return "redirect:/products"; // 목록 페이지로 리다이렉트
    }
}
```

### @RestController 예제 (데이터 반환)

```java
@RestController
@RequestMapping("/api/products")
public class ProductApiController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductApiController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll(); // 객체 목록이 JSON 배열로 변환됨
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok) // 200 OK와 제품 정보 반환
            .orElse(ResponseEntity.notFound().build()); // 404 Not Found 반환
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody @Valid ProductDto productDto) {
        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        
        Product savedProduct = productService.save(product);
        return new ResponseEntity<>(savedProduct, HttpStatus.CREATED); // 201 Created 상태 코드와 함께 생성된 제품 반환
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, 
                                               @RequestBody @Valid ProductDto productDto) {
        return productService.findById(id)
            .map(existingProduct -> {
                existingProduct.setName(productDto.getName());
                existingProduct.setPrice(productDto.getPrice());
                return ResponseEntity.ok(productService.save(existingProduct));
            })
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        return productService.findById(id)
            .map(product -> {
                productService.deleteById(id);
                return ResponseEntity.noContent().<Void>build(); // 204 No Content 반환
            })
            .orElse(ResponseEntity.notFound().build());
    }
}
```

## 같은 컨트롤러에서 뷰와 REST 응답 혼합하기

때로는 하나의 컨트롤러에서 일부 메서드는 뷰를 반환하고, 다른 메서드는 데이터를 반환해야 할 수 있습니다. 이런 경우 `@Controller`를 사용하고 데이터를 반환하는 메서드에만 `@ResponseBody`를 추가할 수 있습니다.

```java
@Controller
public class HybridController {
    
    // HTML 뷰 반환
    @GetMapping("/dashboard")
    public String dashboard(Model model) {
        model.addAttribute("stats", statisticsService.getSummary());
        return "dashboard";
    }
    
    // JSON 데이터 반환
    @GetMapping("/api/stats")
    @ResponseBody
    public Statistics getStatistics() {
        return statisticsService.getSummary();
    }
}
```

## 어떤 것을 선택해야 할까?

### @Controller를 선택해야 하는 경우

- 서버 사이드 렌더링을 사용하는 전통적인 웹 애플리케이션 개발
- 템플릿 엔진(Thymeleaf, JSP 등)을 사용하여 동적 HTML 생성
- 폼 제출 및 유효성 검사가 많은 애플리케이션
- 페이지 간 리다이렉션이 필요한 경우

### @RestController를 선택해야 하는 경우

- RESTful API 개발
- 프론트엔드(React, Angular, Vue.js 등)와 백엔드가 분리된 아키텍처
- 모바일 애플리케이션을 위한 백엔드 API
- 마이크로서비스 아키텍처에서 서비스 간 통신
- JSON/XML 형태의 데이터 교환이 주요 목적인 경우

## 결론

`@Controller`와 `@RestController`는 스프링 MVC에서 웹 요청을 처리하는 두 가지 주요 방식을 나타냅니다. `@Controller`는 전통적인 웹 애플리케이션에서 뷰를 반환하는 데 중점을 두고, `@RestController`는 RESTful 웹 서비스에서 데이터를 직접 반환하는 데 특화되어 있습니다.

애플리케이션의 요구사항과 아키텍처에 따라 적절한 컨트롤러 유형을 선택하는 것이 중요합니다. 서버 사이드 렌더링이 필요한 경우 `@Controller`를, API 엔드포인트를 구현하는 경우 `@RestController`를 사용하는 것이 일반적입니다. 두 가지 기능이 모두 필요한 경우, `@Controller`와 `@ResponseBody`를 조합하여 사용할 수도 있습니다.

다음 포스팅에서는 스프링 부트에서 데이터 유효성 검사(Validation)를 구현하는 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - @Controller](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-controller)
- [Spring Framework 공식 문서 - @RestController](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-restcontroller)
- [Baeldung - @Controller vs @RestController](https://www.baeldung.com/spring-controller-vs-restcontroller)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition)
- [Spring Boot in Action](https://www.manning.com/books/spring-boot-in-action) 