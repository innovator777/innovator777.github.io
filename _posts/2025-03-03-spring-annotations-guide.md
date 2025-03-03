---
layout: post
title: Spring 주요 어노테이션 활용법 총정리
date: 2025-03-03
categories: [Spring Boot]
tags: [spring, spring boot, annotations, 어노테이션, 스프링 어노테이션]
---

이전 포스팅에서는 Bean의 생명주기와 범위에 대해 알아보았습니다. 이번에는 스프링 프레임워크와 스프링 부트에서 자주 사용되는 주요 어노테이션들의 활용법에 대해 자세히 알아보겠습니다. 어노테이션은 스프링 애플리케이션 개발에서 핵심적인 역할을 하며, 이를 잘 이해하고 활용하면 더 효율적이고 간결한 코드를 작성할 수 있습니다.

## 스프링 핵심 어노테이션

### 1. 컴포넌트 등록 관련 어노테이션

#### @Component

스프링 컨테이너에 빈으로 등록할 클래스를 지정하는 가장 기본적인 어노테이션입니다.

```java
@Component
public class EmailService {
    // 이메일 서비스 로직
}
```

#### @Service, @Repository, @Controller, @RestController

`@Component`의 특수화된 형태로, 각각 특정 계층이나 역할을 나타냅니다.

```java
@Service
public class UserService {
    // 비즈니스 로직
}

@Repository
public class UserRepository {
    // 데이터 액세스 로직
}

@Controller
public class UserController {
    // MVC 컨트롤러
}

@RestController
public class UserApiController {
    // REST API 컨트롤러
}
```

#### @Configuration

자바 기반 설정 클래스를 정의할 때 사용합니다.

```java
@Configuration
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        // 데이터 소스 설정 및 반환
    }
    
    @Bean
    public TransactionManager transactionManager(DataSource dataSource) {
        // 트랜잭션 매니저 설정 및 반환
    }
}
```

### 2. 의존성 주입 관련 어노테이션

#### @Autowired

스프링이 적절한 빈을 찾아 자동으로 주입하도록 지시합니다.

```java
@Service
public class OrderService {
    
    private final ProductRepository productRepository;
    private final PaymentService paymentService;
    
    @Autowired  // 생성자가 하나만 있으면 생략 가능
    public OrderService(ProductRepository productRepository, PaymentService paymentService) {
        this.productRepository = productRepository;
        this.paymentService = paymentService;
    }
}
```

#### @Qualifier

같은 타입의 빈이 여러 개 있을 때 특정 빈을 지정하는 데 사용합니다.

```java
@Service
public class NotificationService {
    
    private final MessageSender messageSender;
    
    @Autowired
    public NotificationService(@Qualifier("emailSender") MessageSender messageSender) {
        this.messageSender = messageSender;
    }
}
```

#### @Primary

같은 타입의 빈이 여러 개 있을 때 우선적으로 주입할 빈을 지정합니다.

```java
@Component
@Primary
public class EmailSender implements MessageSender {
    // 이메일 발송 로직
}

@Component
public class SmsSender implements MessageSender {
    // SMS 발송 로직
}
```

#### @Value

프로퍼티 값을 필드에 주입할 때 사용합니다.

```java
@Component
public class DatabaseConfig {
    
    @Value("${database.url}")
    private String url;
    
    @Value("${database.username}")
    private String username;
    
    @Value("${database.password}")
    private String password;
    
    @Value("${app.default.timeout:30}")  // 기본값 30 설정
    private int timeout;
}
```

### 3. 빈 생명주기 관련 어노테이션

#### @PostConstruct

빈 초기화 시 실행할 메서드를 지정합니다.

```java
@Component
public class CacheManager {
    
    private Map<String, Object> cache;
    
    @PostConstruct
    public void init() {
        cache = new ConcurrentHashMap<>();
        System.out.println("캐시 매니저가 초기화되었습니다.");
    }
}
```

#### @PreDestroy

빈 소멸 전에 실행할 메서드를 지정합니다.

```java
@Component
public class ResourceManager {
    
    @PreDestroy
    public void cleanup() {
        System.out.println("리소스 정리를 시작합니다.");
        // 리소스 정리 로직
    }
}
```

### 4. 스코프 관련 어노테이션

#### @Scope

빈의 범위를 지정합니다.

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // 매번 새로운 인스턴스가 생성됨
}

@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedBean {
    // HTTP 세션마다 하나의 인스턴스가 생성됨
}
```

## 스프링 MVC 어노테이션

### 1. 요청 매핑 관련 어노테이션

#### @RequestMapping

HTTP 요청을 특정 메서드에 매핑합니다.

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    @RequestMapping(method = RequestMethod.GET)
    public String listUsers(Model model) {
        // 사용자 목록 조회 로직
        return "user/list";
    }
    
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable Long id, Model model) {
        // 특정 사용자 조회 로직
        return "user/detail";
    }
}
```

#### @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping

`@RequestMapping`의 특수화된 형태로, 각각 특정 HTTP 메서드에 대한 요청을 매핑합니다.

```java
@RestController
@RequestMapping("/api/products")
public class ProductApiController {
    
    @GetMapping
    public List<Product> getAllProducts() {
        // 모든 상품 조회 로직
    }
    
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        // 특정 상품 조회 로직
    }
    
    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        // 상품 생성 로직
    }
    
    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, @RequestBody Product product) {
        // 상품 전체 업데이트 로직
    }
    
    @PatchMapping("/{id}")
    public Product partialUpdateProduct(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        // 상품 부분 업데이트 로직
    }
    
    @DeleteMapping("/{id}")
    public void deleteProduct(@PathVariable Long id) {
        // 상품 삭제 로직
    }
}
```

### 2. 요청 파라미터 관련 어노테이션

#### @PathVariable

URL 경로 변수를 메서드 파라미터에 바인딩합니다.

```java
@GetMapping("/users/{id}/posts/{postId}")
public String getUserPost(@PathVariable Long id, @PathVariable Long postId) {
    // id와 postId를 사용한 로직
}
```

#### @RequestParam

쿼리 파라미터나 폼 데이터를 메서드 파라미터에 바인딩합니다.

```java
@GetMapping("/search")
public String searchProducts(
    @RequestParam String keyword,
    @RequestParam(defaultValue = "1") int page,
    @RequestParam(required = false) String category,
    Model model
) {
    // 검색 로직
}
```

#### @RequestBody

HTTP 요청 본문을 자바 객체로 변환합니다.

```java
@PostMapping("/api/orders")
public ResponseEntity<Order> createOrder(@RequestBody OrderRequest orderRequest) {
    // 주문 생성 로직
}
```

#### @RequestHeader

HTTP 요청 헤더를 메서드 파라미터에 바인딩합니다.

```java
@GetMapping("/api/version")
public String getApiVersion(@RequestHeader("API-Version") String apiVersion) {
    // API 버전에 따른 로직
}
```

### 3. 응답 관련 어노테이션

#### @ResponseBody

메서드 반환값을 HTTP 응답 본문으로 변환합니다.

```java
@Controller
public class DataController {
    
    @GetMapping("/data")
    @ResponseBody
    public Map<String, Object> getData() {
        Map<String, Object> data = new HashMap<>();
        data.put("key", "value");
        return data;  // JSON으로 변환되어 응답됨
    }
}
```

#### @ResponseStatus

HTTP 응답 상태 코드를 지정합니다.

```java
@PostMapping("/api/users")
@ResponseStatus(HttpStatus.CREATED)
public User createUser(@RequestBody User user) {
    // 사용자 생성 로직
}
```

## 스프링 부트 어노테이션

### 1. 애플리케이션 설정 관련 어노테이션

#### @SpringBootApplication

스프링 부트 애플리케이션의 진입점을 표시하며, `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`을 포함합니다.

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

#### @EnableAutoConfiguration

스프링 부트의 자동 구성 기능을 활성화합니다.

```java
@Configuration
@EnableAutoConfiguration
public class AppConfig {
    // 설정 코드
}
```

#### @ConfigurationProperties

외부 설정 프로퍼티를 자바 객체에 바인딩합니다.

```java
@Component
@ConfigurationProperties(prefix = "mail")
public class MailProperties {
    
    private String host;
    private int port;
    private String username;
    private String password;
    private boolean ssl;
    
    // getter와 setter 메서드
}
```

### 2. 테스트 관련 어노테이션

#### @SpringBootTest

스프링 부트 애플리케이션 컨텍스트를 로드하여 통합 테스트를 수행합니다.

```java
@SpringBootTest
public class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    public void testCreateUser() {
        // 테스트 코드
    }
}
```

#### @WebMvcTest

MVC 컨트롤러를 테스트하기 위한 어노테이션입니다.

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    public void testGetUser() throws Exception {
        // given
        given(userService.findById(1L)).willReturn(new User(1L, "John"));
        
        // when & then
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John"));
    }
}
```

#### @DataJpaTest

JPA 컴포넌트를 테스트하기 위한 어노테이션입니다.

```java
@DataJpaTest
public class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    public void testFindByEmail() {
        // given
        User user = new User();
        user.setEmail("test@example.com");
        userRepository.save(user);
        
        // when
        User found = userRepository.findByEmail("test@example.com");
        
        // then
        assertNotNull(found);
        assertEquals("test@example.com", found.getEmail());
    }
}
```

## 스프링 데이터 JPA 어노테이션

### 1. 엔티티 관련 어노테이션

#### @Entity

JPA 엔티티 클래스를 정의합니다.

```java
@Entity
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    private BigDecimal price;
    
    // getter와 setter 메서드
}
```

#### @Table

엔티티와 매핑할 데이터베이스 테이블을 지정합니다.

```java
@Entity
@Table(name = "users", uniqueConstraints = {
    @UniqueConstraint(columnNames = "email")
})
public class User {
    // 필드 및 메서드
}
```

#### @Column

필드와 매핑할 데이터베이스 컬럼을 지정합니다.

```java
@Entity
public class Customer {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "full_name", length = 100, nullable = false)
    private String name;
    
    @Column(unique = true)
    private String email;
    
    // getter와 setter 메서드
}
```

### 2. 관계 매핑 어노테이션

#### @OneToMany, @ManyToOne, @OneToOne, @ManyToMany

엔티티 간의 관계를 정의합니다.

```java
@Entity
public class Order {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    // getter와 setter 메서드
}

@Entity
public class OrderItem {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne
    @JoinColumn(name = "product_id")
    private Product product;
    
    private int quantity;
    
    // getter와 setter 메서드
}
```

## 스프링 시큐리티 어노테이션

### 1. 보안 설정 관련 어노테이션

#### @EnableWebSecurity

웹 보안 설정을 활성화합니다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }
}
```

### 2. 메서드 보안 관련 어노테이션

#### @PreAuthorize, @PostAuthorize

메서드 실행 전/후에 보안 제약 조건을 검사합니다.

```java
@Service
public class AdminService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // 사용자 삭제 로직
    }
    
    @PostAuthorize("returnObject.owner == authentication.name")
    public Document getDocument(Long documentId) {
        // 문서 조회 로직
    }
}
```

#### @Secured

메서드에 대한 접근 권한을 지정합니다.

```java
@Service
public class UserService {
    
    @Secured("ROLE_ADMIN")
    public List<User> getAllUsers() {
        // 모든 사용자 조회 로직
    }
}
```

## 실전 활용 예제

### 1. REST API 컨트롤러 구현

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public List<ProductDTO> getAllProducts(
        @RequestParam(required = false) String category,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        return productService.findProducts(category, page, size);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProductDTO createProduct(@RequestBody @Valid ProductDTO productDTO) {
        return productService.createProduct(productDTO);
    }
    
    @PutMapping("/{id}")
    public ProductDTO updateProduct(
        @PathVariable Long id,
        @RequestBody @Valid ProductDTO productDTO
    ) {
        return productService.updateProduct(id, productDTO);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
    }
}
```

### 2. 설정 프로퍼티 바인딩

```java
@Configuration
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    
    @NotNull
    private String name;
    
    private String description;
    
    @Min(1)
    @Max(100)
    private int maxConnections = 10;
    
    private Security security = new Security();
    
    public static class Security {
        private boolean enabled = true;
        private String tokenSecret;
        private long tokenExpirationMs = 3600000; // 1시간
        
        // getter와 setter 메서드
    }
    
    // getter와 setter 메서드
}
```

### 3. 예외 처리

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFoundException(ResourceNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return new ErrorResponse("VALIDATION_FAILED", "입력값 검증에 실패했습니다.", errors);
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleAllUncaughtException(Exception ex) {
        return new ErrorResponse("INTERNAL_SERVER_ERROR", "서버 오류가 발생했습니다.");
    }
    
    // ErrorResponse 내부 클래스
    @Getter
    @Setter
    public static class ErrorResponse {
        private String code;
        private String message;
        private Map<String, String> errors;
        private LocalDateTime timestamp = LocalDateTime.now();
        
        public ErrorResponse(String code, String message) {
            this.code = code;
            this.message = message;
        }
        
        public ErrorResponse(String code, String message, Map<String, String> errors) {
            this.code = code;
            this.message = message;
            this.errors = errors;
        }
    }
}
```

## 결론

스프링 프레임워크와 스프링 부트는 다양한 어노테이션을 제공하여 개발자가 보다 간결하고 선언적인 방식으로 애플리케이션을 구성할 수 있게 해줍니다. 이러한 어노테이션들은 코드의 가독성을 높이고, 반복적인 설정 코드를 줄이며, 개발 생산성을 크게 향상시킵니다.

이 포스팅에서 다룬 주요 어노테이션들을 잘 이해하고 적절히 활용하면, 스프링의 강력한 기능을 최대한 활용하면서도 유지보수하기 쉬운 애플리케이션을 개발할 수 있습니다.

다음 포스팅에서는 스프링 부트의 자동 구성(Auto Configuration) 메커니즘에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Data JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Spring Security 공식 문서](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
- [Baeldung - Spring 어노테이션](https://www.baeldung.com/spring-annotations)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 