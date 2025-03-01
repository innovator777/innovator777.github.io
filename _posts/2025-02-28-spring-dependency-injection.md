---
layout: post
title: Spring 의존성 주입(DI) 상세 가이드
date: 2025-02-28
categories: [Spring Boot]
tags: [spring, spring boot, dependency injection, di, 의존성 주입]
---

이전 포스팅에서 IoC 컨테이너에 대해 알아보았습니다. 이번에는 IoC의 구체적인 구현 방법인 의존성 주입(Dependency Injection, DI)에 대해 더 자세히 알아보겠습니다. 의존성 주입은 스프링 프레임워크의 핵심 기능 중 하나로, 객체 간의 결합도를 낮추고 코드의 재사용성과 테스트 용이성을 높이는 중요한 디자인 패턴입니다.

## 의존성 주입이란?

의존성 주입은 한 객체가 다른 객체에 의존할 때, 이 의존성을 외부에서 제공(주입)하는 패턴입니다. 객체가 스스로 의존성을 생성하거나 관리하는 대신, 외부(주로 IoC 컨테이너)에서 의존성을 제공받습니다.

### 의존성 주입 없는 코드 vs 의존성 주입 사용 코드

**의존성 주입 없는 코드**:

```java
public class OrderService {
    private PaymentProcessor paymentProcessor;
    
    public OrderService() {
        // 직접 의존성 생성 - 강한 결합
        this.paymentProcessor = new CreditCardProcessor();
    }
    
    public void processOrder(Order order) {
        // 주문 처리 로직
        paymentProcessor.processPayment(order.getAmount());
    }
}
```

**의존성 주입 사용 코드**:

```java
public class OrderService {
    private final PaymentProcessor paymentProcessor;
    
    // 외부에서 의존성 주입 - 느슨한 결합
    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    
    public void processOrder(Order order) {
        // 주문 처리 로직
        paymentProcessor.processPayment(order.getAmount());
    }
}
```

## 의존성 주입의 장점

1. **느슨한 결합도(Loose Coupling)**
   - 객체 간의 의존성이 줄어들어 코드 변경이 용이
   - 한 클래스의 변경이 다른 클래스에 미치는 영향 최소화

2. **테스트 용이성(Testability)**
   - 의존성을 쉽게 모킹(mocking)할 수 있어 단위 테스트가 간편
   - 테스트 더블(Test Double)을 사용한 격리된 테스트 가능

3. **코드 재사용성(Reusability)**
   - 동일한 인터페이스를 구현한 다른 구현체로 쉽게 교체 가능
   - 다양한 환경에서 재사용 가능한 컴포넌트 생성

4. **유지보수성(Maintainability)**
   - 코드의 가독성과 이해도 향상
   - 관심사의 분리(Separation of Concerns) 촉진

5. **확장성(Extensibility)**
   - 기존 코드 수정 없이 새로운 기능 추가 용이
   - 개방-폐쇄 원칙(OCP) 준수 용이

## 스프링에서의 의존성 주입 방법

스프링 프레임워크에서는 세 가지 주요 의존성 주입 방법을 제공합니다.

### 1. 생성자 주입(Constructor Injection)

생성자를 통해 의존성을 주입하는 방식으로, 스프링 팀에서 권장하는 방법입니다.

```java
@Service
public class ProductService {
    private final ProductRepository productRepository;
    private final PriceCalculator priceCalculator;
    
    // 생성자 주입
    public ProductService(ProductRepository productRepository, 
                          PriceCalculator priceCalculator) {
        this.productRepository = productRepository;
        this.priceCalculator = priceCalculator;
    }
    
    // 비즈니스 로직
}
```

**장점**:
- 필수 의존성을 명확하게 표현
- `final` 필드 선언 가능 (불변성 보장)
- 순환 참조 감지 가능 (컴파일 시점)
- 테스트 용이성
- 스프링 4.3부터는 단일 생성자의 경우 `@Autowired` 생략 가능

### 2. 세터 주입(Setter Injection)

세터 메서드를 통해 의존성을 주입하는 방식입니다.

```java
@Service
public class CustomerService {
    private CustomerRepository customerRepository;
    private EmailService emailService;
    
    // 세터 주입
    @Autowired
    public void setCustomerRepository(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    // 비즈니스 로직
}
```

**장점**:
- 선택적 의존성에 적합
- 런타임에 의존성 변경 가능
- 순환 참조 문제 해결 가능

**단점**:
- 의존성이 불변이 아님
- NullPointerException 발생 가능성
- 의존성이 명확하게 표현되지 않음

### 3. 필드 주입(Field Injection)

필드에 직접 의존성을 주입하는 방식입니다. 간결하지만 여러 단점이 있어 권장되지 않습니다.

```java
@Service
public class OrderService {
    // 필드 주입
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private PaymentService paymentService;
    
    // 비즈니스 로직
}
```

**장점**:
- 코드가 간결함
- 설정이 간단함

**단점**:
- 단위 테스트 어려움
- 의존성이 숨겨짐 (투명성 부족)
- final 필드로 선언 불가
- 순환 참조 감지 어려움

## 생성자 주입을 권장하는 이유

스프링 팀은 생성자 주입 방식을 권장합니다. 그 이유는 다음과 같습니다:

1. **불변성(Immutability)**
   - final 필드로 선언하여 객체 불변성 보장
   - 의존성이 변경될 가능성 차단

2. **필수 의존성 명확화**
   - 필수 의존성을 명확하게 표현
   - 객체 생성 시점에 모든 의존성 주입 보장

3. **테스트 용이성**
   - 단위 테스트 시 의존성을 쉽게 모킹 가능
   - 테스트 코드에서 명시적으로 의존성 제공

4. **순환 참조 방지**
   - 컴파일 시점에 순환 참조 감지 가능
   - 애플리케이션 시작 시 즉시 오류 발생

5. **스프링 컨테이너 의존성 제거**
   - 일반 자바 코드로도 인스턴스화 가능
   - 스프링 컨테이너 없이도 단위 테스트 가능

## 다양한 의존성 주입 시나리오

### 인터페이스 기반 의존성 주입

인터페이스를 사용하여 구현체 간의 결합도를 낮추는 방식입니다.

```java
// 인터페이스
public interface NotificationService {
    void sendNotification(String message, String recipient);
}

// 구현체 1
@Service
@Primary
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // 이메일 발송 로직
    }
}

// 구현체 2
@Service
@Qualifier("sms")
public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // SMS 발송 로직
    }
}

// 의존성 주입
@Service
public class UserService {
    private final NotificationService notificationService;
    
    public UserService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
    
    // 기본적으로 @Primary가 지정된 EmailNotificationService 사용
}

@Service
public class AlertService {
    private final NotificationService notificationService;
    
    public AlertService(@Qualifier("sms") NotificationService notificationService) {
        this.notificationService = notificationService;
    }
    
    // @Qualifier로 지정된 SmsNotificationService 사용
}
```

### 컬렉션 의존성 주입

동일한 타입의 여러 빈을 컬렉션으로 주입받는 방식입니다.

```java
@Service
public class NotificationOrchestrator {
    private final List<NotificationService> notificationServices;
    
    public NotificationOrchestrator(List<NotificationService> notificationServices) {
        this.notificationServices = notificationServices;
    }
    
    public void broadcastNotification(String message, String recipient) {
        // 모든 알림 서비스를 통해 메시지 발송
        notificationServices.forEach(service -> 
            service.sendNotification(message, recipient));
    }
}
```

### 조건부 의존성 주입

특정 조건에 따라 의존성을 주입하는 방식입니다.

```java
@Configuration
public class PaymentConfig {
    
    @Bean
    @ConditionalOnProperty(name = "payment.gateway", havingValue = "stripe")
    public PaymentGateway stripePaymentGateway() {
        return new StripePaymentGateway();
    }
    
    @Bean
    @ConditionalOnProperty(name = "payment.gateway", havingValue = "paypal")
    public PaymentGateway paypalPaymentGateway() {
        return new PaypalPaymentGateway();
    }
    
    @Bean
    @ConditionalOnMissingBean(PaymentGateway.class)
    public PaymentGateway defaultPaymentGateway() {
        return new DefaultPaymentGateway();
    }
}
```

## 의존성 주입 관련 어노테이션

스프링에서 의존성 주입과 관련된 주요 어노테이션들을 알아보겠습니다.

### @Autowired

스프링의 의존성 자동 주입을 위한 기본 어노테이션입니다.

```java
@Service
public class ProductService {
    private ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
}
```

### @Qualifier

동일한 타입의 빈이 여러 개 있을 때 특정 빈을 지정하기 위해 사용합니다.

```java
@Service
public class OrderService {
    private final PaymentProcessor paymentProcessor;
    
    public OrderService(@Qualifier("premium") PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}
```

### @Primary

동일한 타입의 빈이 여러 개 있을 때 기본으로 사용할 빈을 지정합니다.

```java
@Service
@Primary
public class StandardPaymentProcessor implements PaymentProcessor {
    // 구현 내용
}
```

### @Value

프로퍼티 값을 주입할 때 사용합니다.

```java
@Service
public class EmailService {
    @Value("${mail.from}")
    private String fromAddress;
    
    @Value("${mail.reply-to:no-reply@example.com}")
    private String replyToAddress;
    
    // 서비스 로직
}
```

### @ConfigurationProperties

타입 안전한 설정 프로퍼티 바인딩을 위해 사용합니다.

```java
@Component
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port;
    private String username;
    private String password;
    private boolean ssl;
    
    // getter, setter
}
```

## 의존성 주입 모범 사례

### 1. 생성자 주입 사용하기

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

### 2. 불변 객체 설계하기

```java
@Service
public class OrderProcessor {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    
    // 모든 의존성을 final로 선언
    public OrderProcessor(OrderRepository orderRepository,
                         PaymentService paymentService,
                         InventoryService inventoryService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

### 3. 인터페이스 기반 설계 활용하기

```java
// 인터페이스 정의
public interface PaymentGateway {
    boolean processPayment(Order order);
}

// 구현체
@Service
public class StripePaymentGateway implements PaymentGateway {
    @Override
    public boolean processPayment(Order order) {
        // Stripe 결제 처리 로직
        return true;
    }
}

// 의존성 주입
@Service
public class CheckoutService {
    private final PaymentGateway paymentGateway;
    
    public CheckoutService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

### 4. 순환 참조 피하기

```java
// 잘못된 예 - 순환 참조
@Service
public class ServiceA {
    private final ServiceB serviceB;
    
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;
    
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}

// 올바른 예 - 공통 인터페이스 추출
public interface CommonService {
    void performCommonOperation();
}

@Service
public class ServiceA implements CommonService {
    private final ServiceB serviceB;
    
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
    
    @Override
    public void performCommonOperation() {
        // 구현
    }
}

@Service
public class ServiceB {
    private final CommonService commonService;
    
    public ServiceB(@Lazy CommonService commonService) {
        this.commonService = commonService;
    }
}
```

### 5. 테스트 가능한 코드 작성하기

```java
@Service
public class ProductService {
    private final ProductRepository productRepository;
    
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }
}

// 테스트 코드
@ExtendWith(MockitoExtension.class)
public class ProductServiceTest {
    @Mock
    private ProductRepository productRepository;
    
    private ProductService productService;
    
    @BeforeEach
    public void setup() {
        productService = new ProductService(productRepository);
    }
    
    @Test
    public void testFindById_whenProductExists() {
        // Given
        Long productId = 1L;
        Product expectedProduct = new Product(productId, "Test Product");
        when(productRepository.findById(productId)).thenReturn(Optional.of(expectedProduct));
        
        // When
        Product actualProduct = productService.findById(productId);
        
        // Then
        assertEquals(expectedProduct.getName(), actualProduct.getName());
        verify(productRepository).findById(productId);
    }
}
```

## 의존성 주입 관련 문제 해결

### 1. NoSuchBeanDefinitionException

빈을 찾을 수 없을 때 발생하는 예외입니다.

**원인**:
- 필요한 빈이 등록되지 않음
- 컴포넌트 스캔 범위에 클래스가 포함되지 않음
- 빈 이름이나 타입이 잘못됨

**해결 방법**:
- `@Component`, `@Service` 등의 어노테이션 확인
- 컴포넌트 스캔 범위 확인
- 빈 이름과 타입 확인
- `@Conditional*` 어노테이션 확인

### 2. 순환 참조(Circular Dependency)

두 빈이 서로를 의존하는 경우 발생하는 문제입니다.

**해결 방법**:
- 설계 재검토 (가장 권장)
- `@Lazy` 어노테이션 사용
- 세터 주입 사용
- 공통 인터페이스 추출

### 3. 다중 빈 정의 문제

동일한 타입의 빈이 여러 개 있을 때 발생하는 문제입니다.

**해결 방법**:
- `@Primary` 어노테이션 사용
- `@Qualifier` 어노테이션 사용
- 이름으로 빈 지정

```java
@Service
public class NotificationService {
    private final MessageSender messageSender;
    
    public NotificationService(@Qualifier("emailSender") MessageSender messageSender) {
        this.messageSender = messageSender;
    }
}
```

## 결론

의존성 주입은 스프링 프레임워크의 핵심 기능으로, 객체 간의 결합도를 낮추고 코드의 재사용성과 테스트 용이성을 높이는 중요한 디자인 패턴입니다. 생성자 주입, 세터 주입, 필드 주입 등 다양한 방법이 있지만, 스프링 팀에서는 불변성과 명확성을 보장하는 생성자 주입을 권장합니다.

의존성 주입을 효과적으로 활용하면 유지보수하기 쉽고, 테스트하기 쉬우며, 확장 가능한 애플리케이션을 개발할 수 있습니다. 인터페이스 기반 설계, 불변 객체 설계, 순환 참조 방지 등의 모범 사례를 따르면 더욱 견고한 애플리케이션을 구축할 수 있습니다.

다음 포스팅에서는 스프링 부트의 자동 구성(Auto Configuration)에 대해 자세히 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - 의존성 주입](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Baeldung - 스프링에서의 생성자 주입](https://www.baeldung.com/constructor-injection-in-spring)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 