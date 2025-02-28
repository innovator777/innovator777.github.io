---
layout: post
title: Spring IoC 컨테이너 이해하기
date: 2025-02-27
categories: [Spring Boot]
tags: [spring, spring boot, ioc, dependency injection, 의존성 주입]
---

스프링 프레임워크의 핵심 개념 중 하나인 IoC(Inversion of Control, 제어의 역전) 컨테이너에 대해 알아보겠습니다. IoC는 스프링의 가장 기본적인 원리이자, 스프링이 다른 프레임워크와 차별화되는 중요한 특징입니다.

## IoC란 무엇인가?

IoC(Inversion of Control)는 직역하면 '제어의 역전'이라는 의미로, 프로그램의 제어 흐름을 개발자가 아닌 프레임워크가 관리하는 디자인 패턴입니다.

### 전통적인 프로그래밍 vs IoC

**전통적인 프로그래밍**:
- 개발자가 필요한 객체를 직접 생성하고 관리
- 객체 간의 의존성을 직접 처리
- 프로그램의 흐름을 개발자가 제어

```java
// 전통적인 방식
public class UserService {
    private UserRepository userRepository;
    
    public UserService() {
        // 개발자가 직접 의존성 객체 생성
        this.userRepository = new UserRepository();
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
}
```

**IoC 방식**:
- 객체의 생성과 생명주기를 프레임워크가 관리
- 의존성을 외부에서 주입받음
- 프로그램의 흐름을 프레임워크가 제어

```java
// IoC 방식
@Service
public class UserService {
    private final UserRepository userRepository;
    
    // 의존성을 외부에서 주입받음 (생성자 주입)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
}
```

### IoC의 장점

1. **느슨한 결합도(Loose Coupling)**: 객체 간의 의존성이 줄어들어 코드 변경이 용이
2. **테스트 용이성**: 의존성을 쉽게 모킹(mocking)할 수 있어 단위 테스트가 쉬움
3. **코드 재사용성 증가**: 동일한 인터페이스를 구현한 다른 구현체로 쉽게 교체 가능
4. **관심사의 분리**: 객체 생성과 사용의 관심사가 분리됨

## 스프링의 IoC 컨테이너

스프링 프레임워크에서는 IoC를 구현하기 위해 IoC 컨테이너를 제공합니다. 이 컨테이너는 객체의 생성, 설정, 관리 및 생명주기를 담당합니다.

### IoC 컨테이너의 종류

스프링에서는 두 가지 유형의 IoC 컨테이너를 제공합니다:

1. **BeanFactory**: 
   - 스프링의 가장 기본적인 IoC 컨테이너
   - 지연 로딩(Lazy Loading) 방식으로 빈을 초기화
   - 리소스가 제한된 환경에 적합

2. **ApplicationContext**: 
   - BeanFactory의 확장 버전
   - 빈을 미리 로딩(Pre-loading)하는 방식
   - 국제화(i18n) 지원, 이벤트 발행, 애플리케이션 계층 컨텍스트 등 추가 기능 제공
   - 대부분의 스프링 애플리케이션에서 사용 권장

```java
// ApplicationContext 예시
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService userService = context.getBean("userService", UserService.class);
```

### 스프링 부트에서의 IoC 컨테이너

스프링 부트에서는 `@SpringBootApplication` 어노테이션을 통해 자동으로 ApplicationContext를 구성합니다.

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(MyApplication.class, args);
        
        // 컨테이너에서 빈 가져오기
        UserService userService = context.getBean(UserService.class);
        userService.findUser(1L);
    }
}
```

## 빈(Bean)이란?

스프링 IoC 컨테이너가 관리하는 객체를 '빈(Bean)'이라고 합니다. 빈은 스프링 컨테이너에 의해 생성, 관리, 제공되는 객체입니다.

### 빈 등록 방법

스프링에서 빈을 등록하는 방법은 여러 가지가 있습니다:

1. **XML 기반 설정**:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userRepository" class="com.example.UserRepository" />
    
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository" />
    </bean>
</beans>
```

2. **자바 기반 설정**:

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserRepository userRepository() {
        return new UserRepository();
    }
    
    @Bean
    public UserService userService() {
        return new UserService(userRepository());
    }
}
```

3. **컴포넌트 스캔**:

```java
@Component // 또는 @Service, @Repository, @Controller 등
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 빈 스코프(Bean Scope)

스프링에서는 빈의 생명주기와 가시성을 정의하는 여러 스코프를 제공합니다:

1. **singleton**: 기본 스코프로, 스프링 컨테이너당 하나의 인스턴스만 생성
2. **prototype**: 요청할 때마다 새로운 인스턴스 생성
3. **request**: HTTP 요청당 하나의 인스턴스 생성 (웹 애플리케이션)
4. **session**: HTTP 세션당 하나의 인스턴스 생성 (웹 애플리케이션)
5. **application**: ServletContext당 하나의 인스턴스 생성 (웹 애플리케이션)
6. **websocket**: WebSocket당 하나의 인스턴스 생성 (웹 애플리케이션)

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // ...
}
```

## 의존성 주입(DI)

의존성 주입(Dependency Injection)은 IoC의 구체적인 구현 방법으로, 객체가 필요로 하는 의존성을 외부에서 주입받는 패턴입니다.

### 의존성 주입 방법

스프링에서는 세 가지 주요 의존성 주입 방법을 제공합니다:

1. **생성자 주입(Constructor Injection)** - 권장 방식:

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    // 생성자 주입
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

2. **세터 주입(Setter Injection)**:

```java
@Service
public class UserService {
    private UserRepository userRepository;
    
    // 세터 주입
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

3. **필드 주입(Field Injection)** - 테스트가 어려워 권장하지 않음:

```java
@Service
public class UserService {
    // 필드 주입
    @Autowired
    private UserRepository userRepository;
}
```

### 생성자 주입을 권장하는 이유

1. **불변성(Immutability)**: final 필드로 선언하여 객체 불변성 보장
2. **필수 의존성 명확화**: 필수 의존성을 명확하게 표현
3. **테스트 용이성**: 단위 테스트 시 의존성을 쉽게 모킹 가능
4. **순환 참조 방지**: 컴파일 시점에 순환 참조 감지 가능

## IoC 컨테이너의 동작 원리

스프링 IoC 컨테이너의 동작 과정을 단계별로 살펴보겠습니다:

1. **설정 정보 로딩**: XML, 자바 설정 클래스, 컴포넌트 스캔 등을 통해 빈 정의 로딩
2. **빈 인스턴스 생성**: 빈 정의를 바탕으로 실제 객체 인스턴스 생성
3. **의존성 주입**: 생성된 빈 객체에 필요한 의존성 주입
4. **초기화 콜백**: 빈 초기화 메서드 호출 (`@PostConstruct`, `InitializingBean` 등)
5. **사용**: 애플리케이션에서 빈 사용
6. **소멸 전 콜백**: 컨테이너 종료 시 소멸 메서드 호출 (`@PreDestroy`, `DisposableBean` 등)
7. **소멸**: 빈 소멸

### 빈 생명주기 콜백

빈의 초기화와 소멸 시점에 특정 작업을 수행하기 위한 여러 방법이 있습니다:

1. **인터페이스 방식**:

```java
@Component
public class DatabaseClient implements InitializingBean, DisposableBean {
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // 초기화 로직
        System.out.println("데이터베이스 연결 초기화");
    }
    
    @Override
    public void destroy() throws Exception {
        // 소멸 로직
        System.out.println("데이터베이스 연결 종료");
    }
}
```

2. **어노테이션 방식** (권장):

```java
@Component
public class DatabaseClient {
    
    @PostConstruct
    public void init() {
        // 초기화 로직
        System.out.println("데이터베이스 연결 초기화");
    }
    
    @PreDestroy
    public void cleanup() {
        // 소멸 로직
        System.out.println("데이터베이스 연결 종료");
    }
}
```

3. **@Bean 메서드 방식**:

```java
@Configuration
public class AppConfig {
    
    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public DatabaseClient databaseClient() {
        return new DatabaseClient();
    }
}
```

## 실전 예제: IoC 컨테이너 활용

간단한 예제를 통해 IoC 컨테이너의 활용 방법을 알아보겠습니다.

### 계층별 컴포넌트 구성

```java
// 도메인 모델
public class User {
    private Long id;
    private String username;
    private String email;
    
    // 생성자, getter, setter 등
}

// 리포지토리 계층
@Repository
public class UserRepository {
    
    private final Map<Long, User> userMap = new HashMap<>();
    
    public User findById(Long id) {
        return userMap.get(id);
    }
    
    public void save(User user) {
        userMap.put(user.getId(), user);
    }
}

// 서비스 계층
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
    
    public void registerUser(User user) {
        // 비즈니스 로직 처리
        userRepository.save(user);
    }
}

// 컨트롤러 계층
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findUser(id);
    }
    
    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.registerUser(user);
    }
}
```

### 테스트에서의 IoC 활용

IoC의 큰 장점 중 하나는 테스트 용이성입니다. 의존성을 쉽게 모킹할 수 있어 단위 테스트가 간편해집니다.

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    public void testFindUser() {
        // Given
        Long userId = 1L;
        User expectedUser = new User();
        expectedUser.setId(userId);
        expectedUser.setUsername("testuser");
        
        when(userRepository.findById(userId)).thenReturn(expectedUser);
        
        // When
        User actualUser = userService.findUser(userId);
        
        // Then
        assertEquals(expectedUser.getUsername(), actualUser.getUsername());
        verify(userRepository).findById(userId);
    }
}
```

## 스프링 부트에서의 IoC 컨테이너 특징

스프링 부트는 스프링 프레임워크의 IoC 컨테이너를 기반으로 하되, 몇 가지 추가적인 특징을 제공합니다:

1. **자동 구성(Auto-Configuration)**: 클래스패스에 있는 라이브러리를 기반으로 빈을 자동 구성
2. **컴포넌트 스캔 자동화**: `@SpringBootApplication`이 있는 패키지부터 하위 패키지까지 자동 스캔
3. **조건부 빈 등록**: `@ConditionalOn*` 어노테이션을 통한 조건부 빈 등록 지원
4. **프로퍼티 기반 구성**: `application.properties` 또는 `application.yml` 파일을 통한 빈 속성 구성

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
    
    // 필요한 경우 추가 빈 정의
    @Bean
    @ConditionalOnMissingBean
    public CustomService customService() {
        return new CustomServiceImpl();
    }
}
```

## IoC 컨테이너 사용 시 주의사항

IoC 컨테이너를 효과적으로 사용하기 위한 몇 가지 주의사항입니다:

1. **순환 참조 방지**: 두 빈이 서로를 의존하는 순환 참조 상황을 피해야 함
2. **빈 스코프 이해**: 특히 prototype 스코프 빈을 singleton 스코프 빈에 주입할 때 주의
3. **컴포넌트 스캔 범위 관리**: 너무 넓은 범위의 컴포넌트 스캔은 성능 저하 가능
4. **빈 이름 충돌 방지**: 동일한 이름의 빈이 여러 개 정의되지 않도록 주의
5. **생성자 주입 권장**: 의존성 주입 시 생성자 주입 방식 사용 권장

## 결론

IoC 컨테이너는 스프링 프레임워크의 핵심 기능으로, 객체의 생성과 의존성 관리를 개발자 대신 프레임워크가 담당함으로써 많은 이점을 제공합니다. 느슨한 결합도, 테스트 용이성, 코드 재사용성 등의 장점을 통해 더 유지보수하기 쉽고 확장 가능한 애플리케이션을 개발할 수 있습니다.

스프링 부트에서는 이러한 IoC 컨테이너의 기능을 더욱 간편하게 사용할 수 있도록 자동 구성과 컴포넌트 스캔 등의 기능을 제공합니다. IoC의 개념과 원리를 잘 이해하고 활용한다면, 스프링 기반의 애플리케이션 개발 시 큰 도움이 될 것입니다.

다음 포스팅에서는 의존성 주입(DI)에 대해 더 자세히 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - IoC 컨테이너](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Martin Fowler의 IoC 설명](https://martinfowler.com/articles/injection.html)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 