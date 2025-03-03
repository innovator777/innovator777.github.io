---
layout: post
title: Spring Bean 생명주기와 범위(Scope) 이해하기
date: 2025-03-02
categories: [Spring Boot]
tags: [spring, spring boot, bean lifecycle, bean scope, 빈 생명주기, 빈 스코프]
---

이전 포스팅에서는 컴포넌트 스캔의 동작 원리에 대해 알아보았습니다. 이번에는 스프링의 핵심 개념인 Bean의 생명주기(Lifecycle)와 범위(Scope)에 대해 자세히 알아보겠습니다. 이 개념들을 이해하면 스프링 애플리케이션의 동작 방식을 더 깊이 이해하고, 효율적인 애플리케이션을 설계할 수 있습니다.

## Bean이란?

먼저 간단히 복습하자면, 스프링에서 Bean은 스프링 IoC 컨테이너가 관리하는 객체를 의미합니다. 스프링 컨테이너는 Bean의 생성, 구성, 관리의 전체 생명주기를 담당합니다.

## Bean 생명주기(Lifecycle)

스프링 Bean은 생성부터 소멸까지 여러 단계의 생명주기를 거칩니다. 이 생명주기를 이해하고 각 단계에 개입할 수 있다면, 애플리케이션의 리소스 관리와 초기화 로직을 효과적으로 제어할 수 있습니다.

### Bean 생명주기의 주요 단계

1. **인스턴스화(Instantiation)**: 스프링 컨테이너가 Bean의 인스턴스를 생성합니다.
2. **프로퍼티 설정(Populate Properties)**: 의존성 주입을 통해 Bean의 프로퍼티 값을 설정합니다.
3. **초기화 콜백(Initialization Callbacks)**: Bean이 모든 필요한 프로퍼티를 받은 후 초기화 콜백 메서드가 호출됩니다.
4. **사용(In Use)**: Bean이 애플리케이션에서 사용됩니다.
5. **소멸 콜백(Destruction Callbacks)**: 컨테이너가 종료될 때 소멸 콜백 메서드가 호출됩니다.
6. **소멸(Destruction)**: Bean이 소멸되고 가비지 컬렉션의 대상이 됩니다.

### 생명주기 콜백 메서드 구현 방법

스프링은 Bean 생명주기의 초기화와 소멸 단계에 개입할 수 있는 여러 방법을 제공합니다.

#### 1. 인터페이스 구현 방식

`InitializingBean`과 `DisposableBean` 인터페이스를 구현하는 방법입니다.

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.stereotype.Component;

@Component
public class DatabaseService implements InitializingBean, DisposableBean {
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // 초기화 로직 - 데이터베이스 연결 등
        System.out.println("DatabaseService가 초기화되었습니다.");
    }
    
    @Override
    public void destroy() throws Exception {
        // 소멸 로직 - 리소스 정리, 연결 종료 등
        System.out.println("DatabaseService가 소멸됩니다.");
    }
    
    // 서비스 메서드들...
}
```

**장점**: 스프링 프레임워크에 의해 명확하게 정의된 계약
**단점**: 스프링 프레임워크에 코드가 결합됨

#### 2. 어노테이션 방식

`@PostConstruct`와 `@PreDestroy` 어노테이션을 사용하는 방법입니다.

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class CacheService {
    
    @PostConstruct
    public void init() {
        // 초기화 로직 - 캐시 준비 등
        System.out.println("CacheService가 초기화되었습니다.");
    }
    
    @PreDestroy
    public void cleanup() {
        // 소멸 로직 - 캐시 정리 등
        System.out.println("CacheService가 소멸됩니다.");
    }
    
    // 서비스 메서드들...
}
```

**장점**: 스프링에 특화되지 않은 JSR-250 표준 어노테이션 사용
**단점**: 일부 환경(예: Java 9+ 모듈 시스템)에서는 추가 의존성 필요

#### 3. Bean 정의 방식

`@Bean` 어노테이션의 `initMethod`와 `destroyMethod` 속성을 사용하는 방법입니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    
    @Bean(initMethod = "initialize", destroyMethod = "shutdown")
    public FileService fileService() {
        return new FileService();
    }
}

public class FileService {
    
    public void initialize() {
        // 초기화 로직 - 파일 시스템 준비 등
        System.out.println("FileService가 초기화되었습니다.");
    }
    
    public void shutdown() {
        // 소멸 로직 - 열린 파일 닫기 등
        System.out.println("FileService가 소멸됩니다.");
    }
    
    // 서비스 메서드들...
}
```

**장점**: 외부 라이브러리 클래스에도 적용 가능, 메서드 이름 자유롭게 지정
**단점**: 설정 클래스에서 Bean 정의 필요

### 생명주기 콜백 실행 순서

여러 생명주기 콜백 메서드를 함께 사용할 경우, 다음 순서로 실행됩니다:

1. `@PostConstruct` 어노테이션이 붙은 메서드
2. `InitializingBean` 인터페이스의 `afterPropertiesSet()` 메서드
3. `@Bean(initMethod="...")` 으로 지정된 초기화 메서드

소멸 시에는 역순으로 실행됩니다:

1. `@PreDestroy` 어노테이션이 붙은 메서드
2. `DisposableBean` 인터페이스의 `destroy()` 메서드
3. `@Bean(destroyMethod="...")` 으로 지정된 소멸 메서드

### 생명주기 콜백 활용 사례

생명주기 콜백은 다음과 같은 상황에서 유용하게 사용됩니다:

1. **리소스 초기화**: 데이터베이스 연결, 네트워크 소켓 열기, 파일 시스템 접근 등
2. **캐시 준비**: 애플리케이션 시작 시 필요한 데이터 미리 로드
3. **외부 시스템 연결**: 메시지 큐, 외부 API 등과의 연결 설정
4. **리소스 정리**: 사용한 리소스 해제, 연결 종료, 임시 파일 삭제 등

## Bean 범위(Scope)

Bean 범위는 Bean 인스턴스가 생성되고 공유되는 방식을 정의합니다. 스프링은 다양한 Bean 범위를 제공하여 다양한 사용 사례에 맞게 Bean의 생명주기를 관리할 수 있게 합니다.

### 주요 Bean 범위

#### 1. 싱글톤(Singleton) - 기본 범위

컨테이너당 하나의 Bean 인스턴스만 생성하고 공유합니다.

```java
@Component
// 또는 @Scope("singleton")
public class UserService {
    // ...
}
```

**특징**:
- 스프링의 기본 범위
- 상태를 공유하므로 스레드 안전성에 주의
- 메모리 효율적, 초기화 비용 절감

**적합한 사용 사례**:
- 상태가 없는(stateless) 서비스
- 공유 설정 정보
- 무거운 리소스를 관리하는 매니저 클래스

#### 2. 프로토타입(Prototype)

요청할 때마다 새로운 Bean 인스턴스를 생성합니다.

```java
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    // ...
}
```

**특징**:
- 매번 새로운 인스턴스 생성
- 스프링은 생성만 관리하고 소멸은 관리하지 않음
- 상태 유지가 필요한 경우 유용

**적합한 사용 사례**:
- 사용자별 상태 유지 필요(예: 장바구니)
- 스레드 안전성이 필요한 상태 저장 객체
- 매번 새로운 상태로 시작해야 하는 객체

#### 3. 웹 관련 범위

웹 애플리케이션에서 사용할 수 있는 추가 범위입니다.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestLogger {
    private final List<String> logs = new ArrayList<>();
    
    public void log(String message) {
        logs.add(message);
    }
    
    public List<String> getLogs() {
        return new ArrayList<>(logs);
    }
}
```

**주요 웹 범위**:
- **request**: HTTP 요청당 하나의 인스턴스
- **session**: HTTP 세션당 하나의 인스턴스
- **application**: ServletContext당 하나의 인스턴스
- **websocket**: WebSocket 세션당 하나의 인스턴스

**특징**:
- 웹 환경에서만 사용 가능
- 프록시 모드 설정 필요(`proxyMode = ScopedProxyMode.TARGET_CLASS`)
- 웹 요청 컨텍스트에 따라 인스턴스 관리

### 범위 간 의존성 주입

서로 다른 범위의 Bean 간에 의존성 주입이 필요한 경우, 특별한 처리가 필요합니다. 특히 싱글톤 Bean이 프로토타입 Bean에 의존할 때 주의해야 합니다.

#### 문제점

```java
@Component
public class SingletonService {
    // 프로토타입 빈을 주입받지만, SingletonService는 한 번만 생성되므로
    // prototypeBean도 한 번만 주입됨
    private final PrototypeBean prototypeBean;
    
    public SingletonService(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }
    
    public void doSomething() {
        // 항상 같은 prototypeBean 인스턴스 사용
        prototypeBean.process();
    }
}

@Component
@Scope("prototype")
public class PrototypeBean {
    // ...
}
```

#### 해결 방법

1. **ObjectFactory 또는 Provider 사용**

```java
@Component
public class SingletonService {
    private final ObjectFactory<PrototypeBean> prototypeBeanFactory;
    
    public SingletonService(ObjectFactory<PrototypeBean> prototypeBeanFactory) {
        this.prototypeBeanFactory = prototypeBeanFactory;
    }
    
    public void doSomething() {
        // 매번 새로운 PrototypeBean 인스턴스 획득
        PrototypeBean prototypeBean = prototypeBeanFactory.getObject();
        prototypeBean.process();
    }
}
```

2. **스코프 프록시 사용**

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class PrototypeBean {
    // ...
}

@Component
public class SingletonService {
    private final PrototypeBean prototypeBean;
    
    public SingletonService(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }
    
    public void doSomething() {
        // 프록시를 통해 매번 새로운 PrototypeBean 인스턴스에 접근
        prototypeBean.process();
    }
}
```

3. **ApplicationContext 사용**

```java
@Component
public class SingletonService implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
    
    public void doSomething() {
        // 컨텍스트에서 매번 새로운 PrototypeBean 인스턴스 획득
        PrototypeBean prototypeBean = applicationContext.getBean(PrototypeBean.class);
        prototypeBean.process();
    }
}
```

## 실전 예제: Bean 생명주기와 범위 활용

### 데이터베이스 연결 관리

```java
@Component
public class DatabaseConnectionManager {
    private Connection connection;
    
    @Value("${db.url}")
    private String dbUrl;
    
    @Value("${db.username}")
    private String username;
    
    @Value("${db.password}")
    private String password;
    
    @PostConstruct
    public void initialize() {
        try {
            System.out.println("데이터베이스 연결 초기화 중...");
            connection = DriverManager.getConnection(dbUrl, username, password);
            System.out.println("데이터베이스 연결 성공!");
        } catch (SQLException e) {
            throw new RuntimeException("데이터베이스 연결 실패", e);
        }
    }
    
    @PreDestroy
    public void cleanup() {
        try {
            if (connection != null && !connection.isClosed()) {
                System.out.println("데이터베이스 연결 종료 중...");
                connection.close();
                System.out.println("데이터베이스 연결 종료 완료!");
            }
        } catch (SQLException e) {
            System.err.println("데이터베이스 연결 종료 중 오류 발생: " + e.getMessage());
        }
    }
    
    public Connection getConnection() {
        return connection;
    }
}
```

### 사용자별 장바구니 관리

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ShoppingCart implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private final List<Product> items = new ArrayList<>();
    
    public void addItem(Product product) {
        items.add(product);
        System.out.println("장바구니에 상품 추가: " + product.getName());
    }
    
    public List<Product> getItems() {
        return new ArrayList<>(items);
    }
    
    public void clear() {
        items.clear();
        System.out.println("장바구니 비우기 완료");
    }
    
    @PostConstruct
    public void init() {
        System.out.println("새로운 장바구니가 생성되었습니다.");
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("장바구니가 소멸됩니다. 포함된 상품 수: " + items.size());
    }
}
```

### 요청 로깅 및 추적

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestTracker {
    private final String requestId = UUID.randomUUID().toString();
    private final long startTime = System.currentTimeMillis();
    private final List<String> operations = new ArrayList<>();
    
    public void trackOperation(String operation) {
        operations.add(operation);
    }
    
    @PostConstruct
    public void init() {
        System.out.println("요청 추적 시작 - ID: " + requestId);
    }
    
    @PreDestroy
    public void logStats() {
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("요청 처리 완료 - ID: " + requestId);
        System.out.println("처리 시간: " + duration + "ms");
        System.out.println("수행된 작업: " + operations.size());
        operations.forEach(op -> System.out.println(" - " + op));
    }
}
```

## 결론

Bean 생명주기와 범위는 스프링 프레임워크의 핵심 개념으로, 이를 잘 이해하고 활용하면 애플리케이션의 리소스 관리와 상태 관리를 효과적으로 수행할 수 있습니다.

생명주기 콜백을 통해 리소스의 초기화와 정리를 적절히 관리하고, 다양한 Bean 범위를 활용하여 애플리케이션의 요구사항에 맞는 상태 관리 전략을 구현할 수 있습니다.

특히 싱글톤과 프로토타입 범위의 특성과 차이점을 이해하고, 웹 애플리케이션에서 request, session 범위를 적절히 활용하면 더 효율적이고 견고한 애플리케이션을 개발할 수 있습니다.

다음 포스팅에서는 스프링 부트의 자동 구성(Auto Configuration) 메커니즘에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - Bean 생명주기](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)
- [Spring Framework 공식 문서 - Bean 범위](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)
- [Baeldung - Spring Bean 생명주기](https://www.baeldung.com/spring-bean-lifecycle)
- [Baeldung - Spring Bean 범위](https://www.baeldung.com/spring-bean-scopes)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 