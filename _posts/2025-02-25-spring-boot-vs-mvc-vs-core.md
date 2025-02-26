---
layout: post
title: 스프링 부트 vs 스프링 MVC vs 스프링 코어
date: 2025-02-25
categories: [Spring Boot]
tags: [spring, spring boot, spring mvc, spring core, java]
---

스프링 생태계는 다양한 모듈과 프로젝트로 구성되어 있습니다. 그 중에서도 스프링 코어, 스프링 MVC, 스프링 부트는 자바 개발자들이 가장 많이 접하는 기술입니다. 이번 포스팅에서는 이 세 가지 기술의 차이점과 각각의 특징에 대해 알아보겠습니다.

## 스프링 코어 (Spring Core)

스프링 코어는 스프링 프레임워크의 기본이 되는 핵심 모듈입니다. 스프링의 가장 근본적인 기능을 제공합니다.

### 주요 특징

1. **IoC (Inversion of Control)**: 객체의 생성과 생명주기 관리를 개발자가 아닌 프레임워크가 담당
2. **DI (Dependency Injection)**: 객체 간의 의존성을 외부에서 주입
3. **AOP (Aspect-Oriented Programming)**: 관점 지향 프로그래밍을 통한 횡단 관심사 분리

### 코드 예시

```java
// 빈 정의
public class UserService {
    private final UserRepository userRepository;
    
    // 생성자 주입 방식의 DI
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
}
```

```xml
<!-- XML 기반 설정 -->
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

```java
// 자바 기반 설정
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

## 스프링 MVC (Spring MVC)

스프링 MVC는 웹 애플리케이션 개발을 위한 모델-뷰-컨트롤러 아키텍처를 구현한 스프링 프레임워크의 모듈입니다.

### 주요 특징

1. **DispatcherServlet**: 모든 요청을 중앙에서 처리하는 프론트 컨트롤러
2. **Handler Mapping**: 요청 URL을 적절한 컨트롤러에 매핑
3. **View Resolver**: 컨트롤러가 반환한 뷰 이름을 실제 뷰로 변환
4. **데이터 바인딩**: HTTP 요청 파라미터를 자바 객체로 자동 변환

### 코드 예시

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public String getUserDetail(@PathVariable Long id, Model model) {
        User user = userService.findUser(id);
        model.addAttribute("user", user);
        return "user/detail";  // 뷰 이름 반환
    }
    
    @PostMapping
    public String createUser(@ModelAttribute User user) {
        userService.saveUser(user);
        return "redirect:/users";
    }
}
```

```xml
<!-- web.xml 설정 -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

## 스프링 부트 (Spring Boot)

스프링 부트는 스프링 프레임워크를 더 쉽게 사용할 수 있도록 만든 프로젝트로, 최소한의 설정으로 스프링 애플리케이션을 빠르게 개발할 수 있게 해줍니다.

### 주요 특징

1. **자동 구성 (Auto Configuration)**: 클래스패스에 있는 라이브러리를 기반으로 자동 설정
2. **스타터 의존성 (Starter Dependencies)**: 특정 기능에 필요한 의존성을 그룹화
3. **내장 서버 (Embedded Server)**: Tomcat, Jetty 등의 서버가 내장되어 별도 설치 불필요
4. **액추에이터 (Actuator)**: 애플리케이션 모니터링 및 관리 기능 제공

### 코드 예시

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {
    
    private final UserService userService;
    
    public UserRestController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findUser(id);
    }
    
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.saveUser(user);
    }
}
```

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

## 주요 차이점 비교

### 설정 방식

**스프링 코어**:
- XML 또는 자바 기반 설정 필요
- 모든 빈과 의존성을 명시적으로 정의
- 세밀한 제어 가능하지만 설정이 복잡

**스프링 MVC**:
- web.xml, servlet-context.xml 등 웹 관련 설정 필요
- 뷰 리졸버, 인터셉터 등 웹 컴포넌트 설정 필요
- URL 매핑 규칙 정의 필요

**스프링 부트**:
- 최소한의 설정으로 시작 가능
- application.properties/yml 파일로 간단히 설정
- 자동 구성으로 대부분의 설정 생략 가능

### 개발 생산성

**스프링 코어**:
- 기본 설정에 많은 시간 소요
- 모든 의존성을 수동으로 관리
- 깊은 이해가 필요

**스프링 MVC**:
- 웹 애플리케이션 구조 설정에 시간 소요
- 서버 설정 및 배포 과정 복잡
- 웹 관련 설정 파일 관리 필요

**스프링 부트**:
- 빠른 프로젝트 시작
- 내장 서버로 즉시 실행 가능
- 스타터 의존성으로 라이브러리 관리 간소화

### 사용 사례

**스프링 코어**:
- 비웹 애플리케이션 (배치, 데스크톱 등)
- 레거시 시스템 통합
- 세밀한 제어가 필요한 엔터프라이즈 애플리케이션

**스프링 MVC**:
- 전통적인 웹 애플리케이션
- JSP, Thymeleaf 등의 템플릿 엔진 기반 뷰
- 복잡한 웹 계층 구조가 필요한 애플리케이션

**스프링 부트**:
- 마이크로서비스 아키텍처
- RESTful API 서버
- 클라우드 네이티브 애플리케이션
- 빠른 개발이 필요한 프로젝트

## 관계 및 사용 패턴

중요한 점은 이 세 가지 기술이 서로 배타적이지 않고 함께 사용된다는 것입니다:

1. **스프링 부트는 스프링 코어와 스프링 MVC를 포함**: 스프링 부트는 스프링 코어와 스프링 MVC의 기능을 모두 사용하면서, 추가적인 편의 기능을 제공합니다.

2. **계층적 관계**: 
   - 스프링 코어: 가장 기본이 되는 계층
   - 스프링 MVC: 스프링 코어 위에 웹 기능 추가
   - 스프링 부트: 스프링 코어와 MVC를 포함하며 자동화 기능 추가

3. **일반적인 현대 개발 패턴**:
   - 스프링 부트를 기반으로 프로젝트 시작
   - 스프링 MVC의 웹 기능 활용
   - 스프링 코어의 IoC/DI 기능 활용

## 기술 선택 가이드

프로젝트 특성에 따른 기술 선택 가이드입니다:

| 프로젝트 유형 | 추천 기술 | 이유 |
|--------------|----------|------|
| 마이크로서비스 | 스프링 부트 | 빠른 개발, 독립적 배포, 클라우드 친화적 |
| 대규모 엔터프라이즈 | 스프링 부트 + 스프링 클라우드 | 확장성, 관리 용이성, 서비스 간 통신 |
| 레거시 시스템 통합 | 스프링 코어 | 유연한 설정, 기존 시스템과의 호환성 |
| 전통적 웹 애플리케이션 | 스프링 MVC | 풍부한 웹 기능, 뷰 템플릿 지원 |
| 간단한 REST API | 스프링 부트 | 빠른 개발, 최소 설정 |

## 결론

스프링 코어, 스프링 MVC, 스프링 부트는 각각 다른 목적과 특징을 가지고 있지만, 함께 사용될 때 가장 강력한 개발 경험을 제공합니다. 현대 자바 웹 개발에서는 스프링 부트를 기반으로 하면서 스프링 MVC의 웹 기능과 스프링 코어의 IoC/DI 기능을 활용하는 방식이 일반적입니다.

프로젝트의 요구사항과 특성에 따라 적절한 조합을 선택하는 것이 중요하며, 스프링 생태계의 다양한 모듈을 이해하고 활용할수록 더 효율적인 개발이 가능해집니다.

다음 포스팅에서는 스프링 부트 프로젝트를 시작하기 위한 JDK와 IDE 설치 및 설정 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Framework 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring MVC 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring 프로젝트 비교 가이드](https://spring.io/projects) 