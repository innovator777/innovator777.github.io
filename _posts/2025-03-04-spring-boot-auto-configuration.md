---
layout: post
title: 스프링 부트 자동 구성(Auto Configuration) 이해하기
date: 2025-03-04
categories: [Spring Boot]
tags: [spring, spring boot, auto configuration, 자동 구성, 스프링 부트 원리]
---

이전 포스팅에서는 스프링의 주요 어노테이션 활용법에 대해 알아보았습니다. 이번에는 스프링 부트의 핵심 기능 중 하나인 자동 구성(Auto Configuration)에 대해 자세히 알아보겠습니다. 자동 구성은 스프링 부트가 기존 스프링 프레임워크와 차별화되는 가장 중요한 특징 중 하나로, 개발자의 설정 부담을 크게 줄여주는 강력한 기능입니다.

## 자동 구성이란?

자동 구성은 스프링 부트가 클래스패스에 있는 라이브러리, 설정, 빈 등을 기반으로 애플리케이션 구성을 자동으로 설정하는 메커니즘입니다. 이를 통해 개발자는 최소한의 설정만으로 애플리케이션을 빠르게 개발할 수 있습니다.

### 전통적인 스프링 vs 스프링 부트

**전통적인 스프링 설정**:
```java
@Configuration
public class WebConfig {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
    
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        return dataSource;
    }
    
    // 더 많은 빈 설정...
}
```

**스프링 부트 설정**:
```properties
# application.properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=password
```

스프링 부트는 `spring-boot-starter-web`과 `spring-boot-starter-data-jpa` 같은 스타터 의존성을 추가하면, 필요한 빈들을 자동으로 구성합니다. 개발자는 프로퍼티 파일에서 기본값을 재정의하는 것만으로 설정을 완료할 수 있습니다.

## 자동 구성의 동작 원리

스프링 부트의 자동 구성은 다음과 같은 단계로 동작합니다.

### 1. @SpringBootApplication 어노테이션

스프링 부트 애플리케이션의 시작점인 `@SpringBootApplication` 어노테이션은 다음 세 가지 어노테이션을 포함합니다:

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {
    // ...
}
```

여기서 자동 구성을 활성화하는 핵심 어노테이션은 `@EnableAutoConfiguration`입니다.

### 2. @EnableAutoConfiguration 어노테이션

이 어노테이션은 `AutoConfigurationImportSelector`를 통해 자동 구성 클래스를 로드합니다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    // ...
}
```

### 3. 자동 구성 클래스 로드

`AutoConfigurationImportSelector`는 `META-INF/spring.factories` 파일에서 `EnableAutoConfiguration` 키에 해당하는 자동 구성 클래스 목록을 로드합니다.

```
# spring.factories 파일 예시
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
# 더 많은 자동 구성 클래스...
```

### 4. 조건부 자동 구성

각 자동 구성 클래스는 `@Conditional` 어노테이션을 사용하여 특정 조건이 충족될 때만 활성화됩니다.

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class,
        DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {
    // ...
}
```

주요 조건부 어노테이션:
- `@ConditionalOnClass`: 특정 클래스가 클래스패스에 있을 때
- `@ConditionalOnMissingClass`: 특정 클래스가 클래스패스에 없을 때
- `@ConditionalOnBean`: 특정 빈이 이미 등록되어 있을 때
- `@ConditionalOnMissingBean`: 특정 빈이 등록되어 있지 않을 때
- `@ConditionalOnProperty`: 특정 프로퍼티가 설정되어 있을 때
- `@ConditionalOnWebApplication`: 웹 애플리케이션일 때
- `@ConditionalOnNotWebApplication`: 웹 애플리케이션이 아닐 때

### 5. 자동 구성 순서

자동 구성 클래스는 `@AutoConfigureOrder`, `@AutoConfigureBefore`, `@AutoConfigureAfter` 어노테이션을 통해 순서를 제어할 수 있습니다.

```java
@Configuration
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class JpaRepositoriesAutoConfiguration {
    // ...
}
```

## 자동 구성 커스터마이징

스프링 부트의 자동 구성은 유연하게 커스터마이징할 수 있습니다.

### 1. 프로퍼티 기반 커스터마이징

`application.properties` 또는 `application.yml` 파일에서 프로퍼티를 설정하여 자동 구성의 동작을 변경할 수 있습니다.

```properties
# 내장 서버 포트 변경
server.port=8081

# 데이터소스 설정
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=password

# JPA 설정
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# 로깅 레벨 설정
logging.level.org.springframework=INFO
logging.level.com.example=DEBUG
```

### 2. 자동 구성 재정의

자동 구성과 동일한 타입의 빈을 직접 정의하여 자동 구성을 재정의할 수 있습니다.

```java
@Configuration
public class CustomDataSourceConfig {
    
    @Bean
    public DataSource dataSource() {
        // 커스텀 데이터소스 구성
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        dataSource.setMaximumPoolSize(20);
        return dataSource;
    }
}
```

### 3. 자동 구성 비활성화

특정 자동 구성 클래스를 비활성화할 수 있습니다.

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

또는 프로퍼티를 통해 비활성화:

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

## 자동 구성 디버깅

자동 구성의 동작을 이해하고 디버깅하기 위한 방법들이 있습니다.

### 1. 디버그 로그 활성화

애플리케이션 실행 시 `--debug` 옵션을 추가하거나 `application.properties`에 설정을 추가합니다.

```properties
debug=true
```

이렇게 하면 스프링 부트는 자동 구성 보고서를 출력합니다:
- 적용된 자동 구성 (Positive matches)
- 적용되지 않은 자동 구성 (Negative matches)
- 제외된 자동 구성 (Exclusions)
- 조건부 평가 보고서 (Condition evaluation report)

### 2. Actuator 엔드포인트 사용

Spring Boot Actuator를 추가하고 `conditions` 엔드포인트를 활성화하면 런타임에 자동 구성 상태를 확인할 수 있습니다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=conditions
```

`/actuator/conditions` 엔드포인트에 접근하여 자동 구성 조건 평가 결과를 확인할 수 있습니다.

## 커스텀 자동 구성 만들기

자신만의 자동 구성을 만들어 재사용 가능한 스프링 부트 스타터를 개발할 수 있습니다.

### 1. 자동 구성 클래스 작성

```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {
    
    private final MyProperties properties;
    
    public MyAutoConfiguration(MyProperties properties) {
        this.properties = properties;
    }
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        MyService service = new MyService();
        service.setProperty(properties.getProperty());
        return service;
    }
}
```

### 2. 프로퍼티 클래스 작성

```java
@ConfigurationProperties(prefix = "my")
public class MyProperties {
    
    private String property = "default";
    
    // getter와 setter
    public String getProperty() {
        return property;
    }
    
    public void setProperty(String property) {
        this.property = property;
    }
}
```

### 3. spring.factories 파일 생성

`META-INF/spring.factories` 파일을 생성하고 자동 구성 클래스를 등록합니다.

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyAutoConfiguration
```

### 4. 스타터 의존성 구조

일반적인 스프링 부트 스타터 구조:

1. **my-spring-boot-starter**: 메인 스타터 모듈 (의존성만 포함)
2. **my-spring-boot-autoconfigure**: 자동 구성 코드 포함
3. **my-spring-boot-starter-test**: 테스트 지원 (선택적)

## 실제 사용 사례

### 1. 웹 애플리케이션 자동 구성

`spring-boot-starter-web`을 추가하면 다음과 같은 자동 구성이 활성화됩니다:

- `WebMvcAutoConfiguration`: Spring MVC 설정
- `EmbeddedWebServerFactoryCustomizerAutoConfiguration`: 내장 웹 서버 설정
- `ErrorMvcAutoConfiguration`: 오류 처리 설정
- `HttpEncodingAutoConfiguration`: HTTP 인코딩 설정

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        model.addAttribute("user", userService.findById(id));
        return "user/detail";
    }
}
```

별도의 설정 없이도 컨트롤러가 동작합니다.

### 2. 데이터 액세스 자동 구성

`spring-boot-starter-data-jpa`를 추가하면 다음과 같은 자동 구성이 활성화됩니다:

- `DataSourceAutoConfiguration`: 데이터소스 설정
- `HibernateJpaAutoConfiguration`: Hibernate 설정
- `JpaRepositoriesAutoConfiguration`: JPA 리포지토리 설정

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    
    // getter와 setter
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
}
```

최소한의 설정으로 JPA 리포지토리가 동작합니다.

### 3. 보안 자동 구성

`spring-boot-starter-security`를 추가하면 다음과 같은 자동 구성이 활성화됩니다:

- `SecurityAutoConfiguration`: 기본 보안 설정
- `WebSecurityEnablerConfiguration`: 웹 보안 활성화
- `UserDetailsServiceAutoConfiguration`: 사용자 인증 서비스 설정

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

기본 보안 설정을 커스터마이징할 수 있습니다.

## 자동 구성의 장단점

### 장점

1. **개발 생산성 향상**: 반복적인 설정 코드 감소
2. **표준화된 구성**: 검증된 모범 사례 기반 설정
3. **유연한 커스터마이징**: 필요에 따라 재정의 가능
4. **빠른 프로토타이핑**: 최소한의 설정으로 빠르게 시작 가능
5. **모듈화 및 재사용성**: 자체 스타터 개발 가능

### 단점

1. **블랙박스 효과**: 내부 동작 이해가 어려울 수 있음
2. **디버깅 복잡성**: 문제 발생 시 원인 파악이 어려울 수 있음
3. **불필요한 빈 생성**: 사용하지 않는 기능도 자동 구성될 수 있음
4. **학습 곡선**: 자동 구성 메커니즘 이해에 시간 필요

## 결론

스프링 부트의 자동 구성은 개발자의 설정 부담을 크게 줄이고, 애플리케이션 개발 속도를 향상시키는 강력한 기능입니다. "관례가 설정보다 우선한다(Convention over Configuration)"는 원칙을 따르면서도, 필요에 따라 세밀한 커스터마이징이 가능한 유연성을 제공합니다.

자동 구성의 동작 원리를 이해하고, 디버깅 방법과 커스터마이징 방법을 숙지하면, 스프링 부트의 장점을 최대한 활용하면서도 애플리케이션의 요구사항에 맞게 조정할 수 있습니다.

다음 포스팅에서는 스프링 부트의 외부화된 설정(Externalized Configuration)과 프로파일(Profiles) 관리에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Boot 공식 문서 - 자동 구성](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.auto-configuration)
- [Spring Boot 공식 문서 - 조건부 어노테이션](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations)
- [Baeldung - 스프링 부트 자동 구성](https://www.baeldung.com/spring-boot-autoconfiguration)
- [Spring Boot 자동 구성 디버깅](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.debugging)
- [커스텀 스타터 만들기](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.custom-starter) 