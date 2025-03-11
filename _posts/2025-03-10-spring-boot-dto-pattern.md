---
layout: post
title: 스프링 부트에서 DTO 패턴 설계 및 구현 가이드
date: 2025-03-10
categories: [Spring Boot]
tags: [spring, spring boot, dto, data transfer object, 객체 매핑, modelMapper]
---

이전 포스팅에서는 스프링 부트의 예외 처리 전략에 대해 알아보았습니다. 이번에는 스프링 부트 애플리케이션에서 DTO(Data Transfer Object) 패턴을 효과적으로 설계하고 구현하는 방법에 대해 자세히 알아보겠습니다. DTO는 계층 간 데이터 전송을 위한 객체로, 클린 아키텍처와 효율적인 API 설계에 중요한 역할을 합니다.

## DTO란 무엇인가?

DTO(Data Transfer Object)는 프로세스 간 또는 계층 간 데이터를 전달하기 위해 사용되는 객체입니다. 주로 다음과 같은 목적으로 사용됩니다:

1. **데이터 캡슐화**: 클라이언트에게 필요한 데이터만 제공
2. **계층 분리**: 도메인 모델과 표현 모델을 분리
3. **API 버전 관리**: API 변경 시 내부 모델을 보호
4. **유효성 검증**: 입력 데이터의 유효성 검사를 위한 전용 객체 제공
5. **네트워크 최적화**: 필요한 데이터만 전송하여 네트워크 트래픽 감소

## DTO vs 도메인 모델

DTO와 도메인 모델(Entity)은 다음과 같은 차이점이 있습니다:

| 특성 | 도메인 모델(Entity) | DTO |
|------|-------------------|-----|
| 목적 | 비즈니스 로직과 상태 표현 | 데이터 전송 |
| 위치 | 서비스 계층, 영속성 계층 | 컨트롤러 계층, 클라이언트 통신 |
| 의존성 | JPA, 비즈니스 규칙 | 없음 (순수 자바 객체) |
| 변경 빈도 | 비즈니스 요구사항에 따라 변경 | API 요구사항에 따라 변경 |
| 유효성 검증 | 도메인 규칙 검증 | 입력 데이터 형식 검증 |

## DTO 설계 원칙

효과적인 DTO 설계를 위한 몇 가지 원칙을 살펴보겠습니다:

### 1. 목적에 맞는 DTO 설계

각 API 엔드포인트나 사용 사례에 맞는 DTO를 설계하세요. 하나의 엔티티에 대해 여러 DTO가 존재할 수 있습니다.

```java
// 사용자 생성을 위한 DTO
public class UserCreateDto {
    private String username;
    private String email;
    private String password;
    // getters, setters, constructors
}

// 사용자 정보 조회를 위한 DTO
public class UserResponseDto {
    private Long id;
    private String username;
    private String email;
    private LocalDateTime createdAt;
    // getters, setters, constructors
}

// 사용자 정보 수정을 위한 DTO
public class UserUpdateDto {
    private String email;
    private String password;
    // getters, setters, constructors
}
```

### 2. 불변성(Immutability) 고려

가능하면 DTO를 불변 객체로 설계하여 데이터 일관성을 유지하세요.

```java
public final class ProductDto {
    private final Long id;
    private final String name;
    private final BigDecimal price;
    private final String category;
    
    public ProductDto(Long id, String name, BigDecimal price, String category) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.category = category;
    }
    
    // getters only (no setters)
    public Long getId() { return id; }
    public String getName() { return name; }
    public BigDecimal getPrice() { return price; }
    public String getCategory() { return category; }
}
```

### 3. 빌더 패턴 활용

복잡한 DTO의 경우 빌더 패턴을 사용하여 객체 생성을 간소화하세요.

```java
public class OrderDto {
    private final Long id;
    private final Long customerId;
    private final List<OrderItemDto> items;
    private final BigDecimal totalAmount;
    private final String status;
    private final LocalDateTime orderDate;
    
    private OrderDto(Builder builder) {
        this.id = builder.id;
        this.customerId = builder.customerId;
        this.items = builder.items;
        this.totalAmount = builder.totalAmount;
        this.status = builder.status;
        this.orderDate = builder.orderDate;
    }
    
    // getters
    
    public static class Builder {
        private Long id;
        private Long customerId;
        private List<OrderItemDto> items;
        private BigDecimal totalAmount;
        private String status;
        private LocalDateTime orderDate;
        
        public Builder id(Long id) {
            this.id = id;
            return this;
        }
        
        public Builder customerId(Long customerId) {
            this.customerId = customerId;
            return this;
        }
        
        // other setters
        
        public OrderDto build() {
            return new OrderDto(this);
        }
    }
}
```

Lombok을 사용하면 더 간단하게 구현할 수 있습니다:

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderDto {
    private Long id;
    private Long customerId;
    private List<OrderItemDto> items;
    private BigDecimal totalAmount;
    private String status;
    private LocalDateTime orderDate;
}
```

## DTO 변환(매핑) 전략

엔티티와 DTO 간의 변환은 다양한 방법으로 구현할 수 있습니다:

### 1. 수동 매핑

가장 기본적인 방법으로, 직접 변환 코드를 작성합니다.

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public UserResponseDto getUserById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
        
        // 수동 매핑
        return new UserResponseDto(
            user.getId(),
            user.getUsername(),
            user.getEmail(),
            user.getCreatedAt()
        );
    }
    
    public User createUser(UserCreateDto dto) {
        // 수동 매핑
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        
        return userRepository.save(user);
    }
}
```

### 2. 변환 메서드

엔티티나 DTO 클래스 내에 변환 메서드를 구현합니다.

```java
// Entity에 toDto 메서드 추가
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private BigDecimal price;
    private String category;
    
    // getters, setters
    
    public ProductDto toDto() {
        return new ProductDto(id, name, price, category);
    }
}

// DTO에 toEntity 메서드 추가
public class ProductDto {
    private Long id;
    private String name;
    private BigDecimal price;
    private String category;
    
    // getters, setters, constructors
    
    public Product toEntity() {
        Product product = new Product();
        product.setId(this.id);
        product.setName(this.name);
        product.setPrice(this.price);
        product.setCategory(this.category);
        return product;
    }
}
```

### 3. 매퍼 클래스

변환 로직을 별도의 매퍼 클래스로 분리합니다.

```java
@Component
public class UserMapper {
    
    public UserResponseDto toResponseDto(User user) {
        return new UserResponseDto(
            user.getId(),
            user.getUsername(),
            user.getEmail(),
            user.getCreatedAt()
        );
    }
    
    public User toEntity(UserCreateDto dto) {
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        user.setPassword(dto.getPassword()); // 실제로는 인코딩 필요
        return user;
    }
    
    public List<UserResponseDto> toResponseDtoList(List<User> users) {
        return users.stream()
            .map(this::toResponseDto)
            .collect(Collectors.toList());
    }
}
```

### 4. 매핑 라이브러리 사용

ModelMapper, MapStruct 등의 라이브러리를 사용하여 자동 매핑을 구현합니다.

**ModelMapper 예제**:

```java
@Configuration
public class ModelMapperConfig {
    
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}

@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final ModelMapper modelMapper;
    
    @Autowired
    public ProductService(ProductRepository productRepository, ModelMapper modelMapper) {
        this.productRepository = productRepository;
        this.modelMapper = modelMapper;
    }
    
    public ProductDto getProductById(Long id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", "id", id));
        
        return modelMapper.map(product, ProductDto.class);
    }
    
    public List<ProductDto> getAllProducts() {
        List<Product> products = productRepository.findAll();
        
        return products.stream()
            .map(product -> modelMapper.map(product, ProductDto.class))
            .collect(Collectors.toList());
    }
}
```

**MapStruct 예제**:

```java
@Mapper(componentModel = "spring")
public interface ProductMapper {
    
    ProductDto toDto(Product product);
    
    Product toEntity(ProductDto dto);
    
    List<ProductDto> toDtoList(List<Product> products);
}

@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    private final ProductMapper productMapper;
    
    @Autowired
    public ProductService(ProductRepository productRepository, ProductMapper productMapper) {
        this.productRepository = productRepository;
        this.productMapper = productMapper;
    }
    
    public ProductDto getProductById(Long id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", "id", id));
        
        return productMapper.toDto(product);
    }
}
```

## DTO 유효성 검증

Spring Validation을 사용하여 DTO의 유효성을 검증할 수 있습니다.

```java
public class UserCreateDto {
    
    @NotBlank(message = "Username is required")
    @Size(min = 4, max = 50, message = "Username must be between 4 and 50 characters")
    private String username;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$", 
             message = "Password must contain at least one digit, one lowercase, one uppercase, and one special character")
    private String password;
    
    // getters, setters, constructors
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<UserResponseDto> createUser(@Valid @RequestBody UserCreateDto dto) {
        User user = userService.createUser(dto);
        UserResponseDto responseDto = userService.toResponseDto(user);
        return new ResponseEntity<>(responseDto, HttpStatus.CREATED);
    }
}
```

## DTO 설계 모범 사례

### 1. 계층별 DTO 분리

API 계층과 서비스 계층에서 사용하는 DTO를 분리하여 관리하세요.

```
com.example.dto
├── api
│   ├── request
│   │   ├── UserCreateRequest.java
│   │   └── UserUpdateRequest.java
│   └── response
│       ├── UserResponse.java
│       └── UserDetailResponse.java
└── service
    ├── UserCreateDto.java
    └── UserUpdateDto.java
```

### 2. 상속과 합성을 활용한 DTO 설계

공통 속성을 가진 DTO는 상속이나 합성을 통해 재사용하세요.

```java
// 기본 DTO
public abstract class BaseDto {
    private Long id;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // getters, setters
}

// 상속을 통한 확장
public class ProductDto extends BaseDto {
    private String name;
    private BigDecimal price;
    private String category;
    
    // getters, setters
}

// 합성을 통한 확장
public class OrderDto {
    private Long id;
    private CustomerDto customer;
    private List<OrderItemDto> items;
    private BigDecimal totalAmount;
    
    // getters, setters
}
```

### 3. 버전 관리

API 버전 변경 시 DTO도 함께 버전 관리하세요.

```
com.example.dto
├── v1
│   ├── UserDto.java
│   └── ProductDto.java
└── v2
    ├── UserDto.java
    └── ProductDto.java
```

## 결론

DTO 패턴은 계층 간 데이터 전송을 효율적으로 관리하고, 도메인 모델을 외부 노출로부터 보호하는 중요한 역할을 합니다. 목적에 맞는 DTO 설계, 불변성 고려, 효율적인 매핑 전략, 그리고 적절한 유효성 검증을 통해 더 견고하고 유지보수하기 쉬운 애플리케이션을 구축할 수 있습니다.

특히 API 설계에서는 클라이언트의 요구사항에 맞는 DTO를 제공함으로써 불필요한 데이터 전송을 줄이고, 버전 관리를 용이하게 할 수 있습니다. 또한 MapStruct나 ModelMapper와 같은 매핑 라이브러리를 활용하면 반복적인 변환 코드를 줄이고 개발 생산성을 높일 수 있습니다.

다음 포스팅에서는 스프링 부트에서 데이터 유효성 검사(Validation)를 구현하는 방법에 대해 더 자세히 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [MapStruct 공식 문서](https://mapstruct.org/documentation/stable/reference/html/)
- [ModelMapper 공식 문서](http://modelmapper.org/getting-started/)
- [Baeldung - DTO with MapStruct](https://www.baeldung.com/mapstruct)
- [Baeldung - Quick Guide to ModelMapper](https://www.baeldung.com/java-modelmapper-lists)
- [Martin Fowler - DTO Pattern](https://martinfowler.com/eaaCatalog/dataTransferObject.html) 