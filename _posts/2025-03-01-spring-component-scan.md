---
layout: post
title: Spring 컴포넌트 스캔 동작 원리 이해하기
date: 2025-03-01
categories: [Spring Boot]
tags: [spring, spring boot, component scan, 컴포넌트 스캔, 빈 등록]
---

이전 포스팅에서는 IoC 컨테이너와 의존성 주입(DI)에 대해 알아보았습니다. 이번에는 스프링의 또 다른 핵심 기능인 컴포넌트 스캔(Component Scan)의 동작 원리에 대해 자세히 알아보겠습니다. 컴포넌트 스캔은 스프링이 자동으로 빈을 찾고 등록하는 메커니즘으로, 개발자가 수동으로 빈을 등록하는 번거로움을 줄여줍니다.

## 컴포넌트 스캔이란?

컴포넌트 스캔은 스프링이 애플리케이션의 클래스패스에서 특정 어노테이션이 붙은 클래스를 찾아 자동으로 빈으로 등록하는 기능입니다. 이를 통해 개발자는 XML 설정 파일이나 자바 설정 클래스에서 일일이 빈을 정의하지 않아도 됩니다.

## 컴포넌트 스캔의 장점

1. **설정 간소화**: 빈 등록을 위한 XML 또는 자바 설정 코드 감소
2. **개발 생산성 향상**: 새로운 컴포넌트 추가 시 자동으로 빈으로 등록
3. **유지보수성 향상**: 관련 코드와 설정이 분리되지 않고 함께 위치
4. **모듈화 촉진**: 각 컴포넌트가 자신의 역할을 명확히 표현

## 컴포넌트 스캔 활성화 방법

### XML 기반 설정

```xml
<context:component-scan base-package="com.example.myapp" />
```

### 자바 기반 설정

```java
@Configuration
@ComponentScan(basePackages = "com.example.myapp")
public class AppConfig {
    // 추가 설정
}
```

### 스프링 부트

스프링 부트에서는 `@SpringBootApplication` 어노테이션이 자동으로 컴포넌트 스캔을 활성화합니다.

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`@SpringBootApplication`은 내부적으로 `@ComponentScan`을 포함하고 있으며, 기본적으로 해당 클래스가 위치한 패키지와 그 하위 패키지를 스캔 대상으로 합니다.

## 컴포넌트 스캔 대상 어노테이션

스프링은 다음 어노테이션이 붙은 클래스를 컴포넌트 스캔 대상으로 인식합니다:

1. **@Component**: 일반적인 스프링 컴포넌트
2. **@Service**: 비즈니스 로직을 담당하는 서비스 계층 컴포넌트
3. **@Repository**: 데이터 액세스 계층 컴포넌트
4. **@Controller**: 웹 MVC 컨트롤러
5. **@RestController**: REST API 컨트롤러
6. **@Configuration**: 설정 클래스

`@Service`, `@Repository`, `@Controller`, `@RestController`, `@Configuration`은 모두 내부적으로 `@Component`를 포함하고 있습니다. 이러한 특수 어노테이션은 해당 컴포넌트의 역할을 더 명확하게 표현하고, 특정 기능(예: 트랜잭션 관리, 예외 변환 등)을 활성화하는 데 사용됩니다.

```java
@Component
public class SimpleComponent {
    // 일반 컴포넌트
}

@Service
public class UserService {
    // 비즈니스 로직 서비스
}

@Repository
public class UserRepository {
    // 데이터 액세스 컴포넌트
}

@Controller
public class UserController {
    // 웹 MVC 컨트롤러
}
```

## 컴포넌트 스캔의 동작 원리

컴포넌트 스캔의 내부 동작 과정을 단계별로 살펴보겠습니다.

### 1. 스캔 대상 패키지 결정

컴포넌트 스캔은 지정된 베이스 패키지와 그 하위 패키지를 스캔 대상으로 합니다. 스프링 부트에서는 `@SpringBootApplication` 어노테이션이 있는 클래스의 패키지가 기본 베이스 패키지가 됩니다.

### 2. 클래스패스 스캐닝

스프링은 클래스패스 리소스를 스캔하여 지정된 패키지 내의 모든 클래스를 찾습니다. 이 과정에서 ASM 라이브러리를 사용하여 클래스 메타데이터를 분석합니다.

### 3. 후보 컴포넌트 식별

스캔된 클래스 중에서 `@Component` 또는 `@Component`를 메타 어노테이션으로 가진 어노테이션(`@Service`, `@Repository` 등)이 붙은 클래스를 후보 컴포넌트로 식별합니다.

### 4. 필터 적용

컴포넌트 스캔은 포함 필터(include filter)와 제외 필터(exclude filter)를 적용하여 스캔 대상을 세밀하게 제어할 수 있습니다.

```java
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*Service"),
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Deprecated.class)
)
```

### 5. 빈 정의 생성

식별된 각 컴포넌트에 대해 `BeanDefinition`을 생성합니다. `BeanDefinition`은 빈의 메타데이터를 담고 있으며, 스프링 컨테이너가 빈을 생성하고 관리하는 데 필요한 정보를 제공합니다.

### 6. 빈 이름 생성

각 빈에 대해 고유한 이름을 생성합니다. 기본적으로 클래스 이름의 첫 글자를 소문자로 변환한 이름을 사용합니다(예: `UserService` → `userService`). `@Component("customName")` 형태로 명시적인 이름을 지정할 수도 있습니다.

### 7. 빈 등록

생성된 `BeanDefinition`을 스프링 컨테이너에 등록합니다. 이 시점에서는 실제 빈 인스턴스가 생성되지 않고, 빈의 정의만 등록됩니다.

### 8. 빈 생성 및 의존성 주입

애플리케이션 컨텍스트가 초기화될 때, 등록된 빈 정의를 기반으로 실제 빈 인스턴스를 생성하고 의존성을 주입합니다.

## 컴포넌트 스캔 커스터마이징

컴포넌트 스캔의 동작을 다양한 방법으로 커스터마이징할 수 있습니다.

### 스캔 범위 지정

```java
@ComponentScan(
    basePackages = {"com.example.service", "com.example.repository"},
    basePackageClasses = {MarkerService.class, MarkerRepository.class}
)
```

`basePackages`는 문자열로 패키지를 지정하고, `basePackageClasses`는 마커 클래스(또는 인터페이스)를 통해 패키지를 지정합니다. 마커 클래스 방식은 타입 안전성을 제공하므로 리팩토링 시 더 안전합니다.

### 필터 적용

```java
@ComponentScan(
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = CustomComponent.class),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = UserService.class)
    },
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.PATTERN, pattern = "com.example.legacy.*")
    }
)
```

필터 타입:
- `FilterType.ANNOTATION`: 특정 어노테이션이 있는 클래스
- `FilterType.ASSIGNABLE_TYPE`: 특정 타입이나 그 하위 타입
- `FilterType.ASPECTJ`: AspectJ 패턴
- `FilterType.REGEX`: 정규 표현식
- `FilterType.CUSTOM`: 사용자 정의 필터

### 스캔 모드 설정

```java
@ComponentScan(
    lazyInit = true,
    scopedProxy = ScopedProxyMode.INTERFACES
)
```

- `lazyInit`: 빈을 지연 초기화할지 여부
- `scopedProxy`: 스코프 프록시 모드 설정 (프로토타입 빈을 싱글톤 빈에 주입할 때 유용)

## 실전 예제: 컴포넌트 스캔 활용

### 기본 구조 설정

```
com.example.myapp
├── MyApplication.java
├── config
│   └── AppConfig.java
├── controller
│   ├── UserController.java
│   └── ProductController.java
├── service
│   ├── UserService.java
│   └── ProductService.java
└── repository
    ├── UserRepository.java
    └── ProductRepository.java
```

### 애플리케이션 클래스

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 컴포넌트 예제

```java
package com.example.myapp.repository;

import org.springframework.stereotype.Repository;

@Repository
public class UserRepository {
    // 데이터 액세스 로직
}
```

```java
package com.example.myapp.service;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import com.example.myapp.repository.UserRepository;

@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // 비즈니스 로직
}
```

```java
package com.example.myapp.controller;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;
import com.example.myapp.service.UserService;

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // API 엔드포인트
}
```

### 커스텀 컴포넌트 스캔 설정

특정 요구사항에 맞게 컴포넌트 스캔을 커스터마이징하는 예제입니다.

```java
package com.example.myapp.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
    basePackages = "com.example.myapp",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = ".*Service"
    ),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Deprecated.class
    )
)
public class CustomScanConfig {
    // 추가 설정
}
```

## 컴포넌트 스캔 사용 시 주의사항

### 1. 스캔 범위 관리

너무 넓은 범위의 컴포넌트 스캔은 애플리케이션 시작 시간을 늘리고 불필요한 빈을 등록할 수 있습니다. 필요한 패키지만 스캔하도록 범위를 적절히 설정하세요.

### 2. 빈 이름 충돌

동일한 이름의 빈이 여러 개 등록되면 충돌이 발생합니다. 명시적으로 빈 이름을 지정하거나, 패키지 구조를 잘 설계하여 충돌을 방지하세요.

```java
@Component("userManager")  // 명시적 이름 지정
public class UserService {
    // ...
}
```

### 3. 순환 참조

컴포넌트 스캔으로 등록된 빈 사이에 순환 참조가 발생할 수 있습니다. 이는 애플리케이션 시작 시 오류를 발생시키거나 예기치 않은 동작을 유발할 수 있습니다.

### 4. 테스트 환경 설정

테스트 환경에서는 필요한 컴포넌트만 스캔하도록 설정하여 테스트 실행 속도를 향상시키고 테스트 격리성을 유지하세요.

```java
@SpringBootTest(classes = {UserService.class, UserRepository.class})
public class UserServiceTest {
    // 테스트 코드
}
```

## 컴포넌트 스캔 vs 수동 빈 등록

컴포넌트 스캔과 수동 빈 등록은 상호 배타적이지 않으며, 함께 사용할 수 있습니다. 각 방식의 장단점을 이해하고 적절히 조합하는 것이 중요합니다.

### 컴포넌트 스캔 선호 상황

- 표준 애플리케이션 구조를 가진 경우
- 많은 수의 컴포넌트를 등록해야 하는 경우
- 빠른 개발과 간결한 코드를 우선시하는 경우

### 수동 빈 등록 선호 상황

- 제3자 라이브러리의 클래스를 빈으로 등록해야 하는 경우
- 조건부 빈 등록이 필요한 경우
- 빈 생성 과정을 세밀하게 제어해야 하는 경우
- 설정의 중앙화가 필요한 경우

```java
@Configuration
public class ManualConfig {
    @Bean
    public ThirdPartyService thirdPartyService() {
        ThirdPartyService service = new ThirdPartyService();
        service.setProperty("value");
        return service;
    }
    
    @Bean
    @Conditional(DevProfileCondition.class)
    public DataSource devDataSource() {
        // 개발 환경용 데이터 소스
    }
    
    @Bean
    @Conditional(ProdProfileCondition.class)
    public DataSource prodDataSource() {
        // 운영 환경용 데이터 소스
    }
}
```

## 결론

컴포넌트 스캔은 스프링 애플리케이션에서 빈을 자동으로 등록하는 강력한 메커니즘입니다. 이를 통해 개발자는 반복적인 설정 코드를 줄이고 더 선언적인 방식으로 애플리케이션을 구성할 수 있습니다.

컴포넌트 스캔의 동작 원리를 이해하고 적절히 활용하면, 코드의 가독성과 유지보수성을 높이면서도 스프링의 강력한 IoC 기능을 최대한 활용할 수 있습니다. 필요에 따라 스캔 범위를 조정하고, 필터를 적용하며, 수동 빈 등록과 조합하여 최적의 애플리케이션 구조를 설계하세요.

다음 포스팅에서는 스프링 부트의 자동 구성(Auto Configuration) 메커니즘에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서 - 컴포넌트 스캔](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-autodetection)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Baeldung - 스프링 컴포넌트 스캔](https://www.baeldung.com/spring-component-scanning)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 