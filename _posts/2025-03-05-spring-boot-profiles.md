---
layout: post
title: 스프링 부트 프로파일(Profiles) 설정과 활용
date: 2025-03-05
categories: [Spring Boot]
tags: [spring, spring boot, profiles, 프로파일, 환경 설정]
---

이전 포스팅에서는 스프링 부트의 자동 구성(Auto Configuration)에 대해 알아보았습니다. 이번에는 스프링 부트의 또 다른 중요한 기능인 프로파일(Profiles)에 대해 자세히 알아보겠습니다. 프로파일은 서로 다른 환경(개발, 테스트, 운영 등)에 맞게 애플리케이션 설정을 분리하고 관리할 수 있게 해주는 강력한 기능입니다.

## 프로파일이란?

프로파일은 애플리케이션의 설정을 환경별로 구분하여 관리할 수 있게 해주는 메커니즘입니다. 개발, 테스트, 스테이징, 운영 등 다양한 환경에서 서로 다른 설정(데이터베이스 연결 정보, 외부 서비스 URL, 로깅 레벨 등)을 쉽게 전환할 수 있습니다.

## 프로파일 사용의 필요성

실제 애플리케이션 개발 및 운영 과정에서는 다양한 환경에서 서로 다른 설정이 필요합니다:

1. **개발 환경**: 로컬 데이터베이스, 상세한 로깅, 빠른 개발을 위한 설정
2. **테스트 환경**: 테스트용 데이터베이스, 테스트 서버 연결 정보
3. **스테이징 환경**: 운영과 유사하지만 실제 데이터를 사용하지 않는 환경
4. **운영 환경**: 실제 사용자가 접근하는 환경, 보안 강화, 성능 최적화 설정

이러한 다양한 환경에서 코드 변경 없이 설정만 전환하여 애플리케이션을 실행할 수 있다면, 개발과 배포 과정이 훨씬 간소화됩니다.

## 프로파일 설정 방법

### 1. 프로퍼티 파일을 통한 프로파일 설정

스프링 부트에서는 `application-{profile}.properties` 또는 `application-{profile}.yml` 형식의 파일을 통해 프로파일별 설정을 관리할 수 있습니다.

**기본 설정 파일**: `application.properties` 또는 `application.yml`
```properties
# application.properties - 모든 환경에 공통으로 적용되는 설정
spring.application.name=my-application
logging.level.root=INFO
```

**개발 환경 설정**: `application-dev.properties`
```properties
# 개발 환경 설정
spring.datasource.url=jdbc:mysql://localhost:3306/devdb
spring.datasource.username=devuser
spring.datasource.password=devpass
logging.level.com.example=DEBUG
```

**테스트 환경 설정**: `application-test.properties`
```properties
# 테스트 환경 설정
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create-drop
```

**운영 환경 설정**: `application-prod.properties`
```properties
# 운영 환경 설정
spring.datasource.url=jdbc:mysql://prod-db-server:3306/proddb
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
logging.level.root=WARN
logging.level.com.example=INFO
```

### 2. YAML 파일을 사용한 다중 프로파일 설정

YAML 파일에서는 `---` 구분자를 사용하여 하나의 파일 내에 여러 프로파일 설정을 포함할 수 있습니다.

```yaml
# application.yml
spring:
  application:
    name: my-application

logging:
  level:
    root: INFO

---
spring:
  config:
    activate:
      on-profile: dev
  
  datasource:
    url: jdbc:mysql://localhost:3306/devdb
    username: devuser
    password: devpass

logging:
  level:
    com.example: DEBUG

---
spring:
  config:
    activate:
      on-profile: test
  
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password: 
  
  jpa:
    hibernate:
      ddl-auto: create-drop

---
spring:
  config:
    activate:
      on-profile: prod
  
  datasource:
    url: jdbc:mysql://prod-db-server:3306/proddb
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

logging:
  level:
    root: WARN
    com.example: INFO
```

## 프로파일 활성화 방법

프로파일을 활성화하는 방법에는 여러 가지가 있습니다.

### 1. 애플리케이션 속성으로 설정

`application.properties` 또는 `application.yml` 파일에서 활성 프로파일을 지정할 수 있습니다.

```properties
# application.properties
spring.profiles.active=dev
```

```yaml
# application.yml
spring:
  profiles:
    active: dev
```

### 2. 환경 변수 사용

환경 변수를 통해 활성 프로파일을 지정할 수 있습니다.

```bash
# Linux/macOS
export SPRING_PROFILES_ACTIVE=prod

# Windows
set SPRING_PROFILES_ACTIVE=prod
```

### 3. 명령행 인수 사용

애플리케이션 실행 시 명령행 인수로 프로파일을 지정할 수 있습니다.

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

### 4. 프로그래밍 방식으로 설정

`SpringApplication` 클래스를 사용하여 프로그래밍 방식으로 프로파일을 설정할 수 있습니다.

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MyApplication.class);
    app.setAdditionalProfiles("prod");
    app.run(args);
}
```

## 다중 프로파일 활성화

여러 프로파일을 동시에 활성화할 수도 있습니다.

```properties
# 쉼표로 구분하여 여러 프로파일 활성화
spring.profiles.active=prod,monitoring
```

```bash
java -jar myapp.jar --spring.profiles.active=prod,monitoring
```

## 프로파일 그룹 설정 (Spring Boot 2.4+)

스프링 부트 2.4부터는 프로파일 그룹 기능이 추가되어, 여러 프로파일을 하나의 그룹으로 묶어 관리할 수 있습니다.

```properties
# application.properties
spring.profiles.group.production=prod,monitoring,analytics
spring.profiles.group.development=dev,local-db
```

```yaml
# application.yml
spring:
  profiles:
    group:
      production: prod,monitoring,analytics
      development: dev,local-db
```

이제 `production` 또는 `development` 프로파일을 활성화하면, 해당 그룹에 포함된 모든 프로파일이 함께 활성화됩니다.

```bash
java -jar myapp.jar --spring.profiles.active=production
```

## @Profile 어노테이션을 사용한 조건부 빈 등록

`@Profile` 어노테이션을 사용하면 특정 프로파일이 활성화되었을 때만 빈을 등록하도록 설정할 수 있습니다.

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        // 개발 환경용 데이터 소스 설정
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        // 운영 환경용 데이터 소스 설정
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://prod-server:3306/proddb");
        dataSource.setUsername("produser");
        dataSource.setPassword("prodpass");
        return dataSource;
    }
}
```

클래스 레벨에도 `@Profile` 어노테이션을 적용할 수 있습니다.

```java
@Configuration
@Profile("dev")
public class DevOnlyConfiguration {
    // 개발 환경에서만 필요한 빈 설정
}
```

## 프로파일 활용 사례

### 1. 데이터베이스 설정 분리

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }
    
    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("classpath:schema-test.sql")
                .addScript("classpath:data-test.sql")
                .build();
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource(
            @Value("${db.url}") String url,
            @Value("${db.username}") String username,
            @Value("${db.password}") String password) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}
```

### 2. 외부 서비스 연동 설정

```java
@Service
public class PaymentService {
    
    private final PaymentGateway paymentGateway;
    
    public PaymentService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
    
    // 결제 처리 로직
}

@Configuration
public class PaymentConfig {
    
    @Bean
    @Profile("dev")
    public PaymentGateway mockPaymentGateway() {
        return new MockPaymentGateway();
    }
    
    @Bean
    @Profile("prod")
    public PaymentGateway realPaymentGateway(
            @Value("${payment.api.key}") String apiKey,
            @Value("${payment.api.url}") String apiUrl) {
        return new RealPaymentGateway(apiKey, apiUrl);
    }
}
```

### 3. 보안 설정 분리

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    @Profile("dev")
    public SecurityFilterChain devSecurityFilterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeRequests()
                .anyRequest().permitAll()
                .and().csrf().disable()
                .build();
    }
    
    @Bean
    @Profile("prod")
    public SecurityFilterChain prodSecurityFilterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeRequests()
                .antMatchers("/api/**").authenticated()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().permitAll()
                .and().formLogin()
                .and().csrf()
                .build();
    }
}
```

## 프로파일 사용 시 주의사항

1. **민감한 정보 관리**: 운영 환경의 비밀번호, API 키 등은 프로퍼티 파일에 직접 저장하지 말고, 환경 변수나 외부 설정 서버(Spring Cloud Config)를 사용하세요.

2. **기본 프로파일 설정**: 프로파일을 지정하지 않으면 `default` 프로파일이 사용됩니다. 개발 환경에서 실수로 운영 환경에 접근하지 않도록 기본 프로파일을 안전하게 설정하세요.

3. **프로파일 우선순위**: 여러 프로파일이 활성화된 경우, 나중에 로드된 프로파일의 설정이 이전 설정을 덮어씁니다. 이 우선순위를 이해하고 활용하세요.

4. **프로파일 테스트**: 각 프로파일 설정이 제대로 작동하는지 테스트하는 것이 중요합니다. 스프링 부트 테스트에서는 `@ActiveProfiles` 어노테이션을 사용하여 테스트 시 특정 프로파일을 활성화할 수 있습니다.

```java
@SpringBootTest
@ActiveProfiles("test")
public class ApplicationTests {
    // 테스트 코드
}
```

## 결론

스프링 부트의 프로파일 기능은 다양한 환경에서 애플리케이션을 쉽게 구성하고 전환할 수 있게 해주는 강력한 도구입니다. 개발, 테스트, 운영 등 각 환경에 맞는 설정을 분리하여 관리함으로써, 코드 변경 없이 환경에 따라 적절한 설정을 적용할 수 있습니다.

프로파일을 효과적으로 활용하면 환경별 설정 관리가 용이해지고, 배포 프로세스가 간소화되며, 애플리케이션의 유연성과 안정성이 향상됩니다. 특히 마이크로서비스 아키텍처나 클라우드 환경에서는 프로파일을 통한 설정 관리가 더욱 중요해집니다.

다음 포스팅에서는 스프링 부트의 외부화된 설정(Externalized Configuration)과 설정 속성 바인딩에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Boot 공식 문서 - 프로파일](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles)
- [Spring Framework 공식 문서 - 환경 추상화](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-environment)
- [Baeldung - 스프링 프로파일](https://www.baeldung.com/spring-profiles)
- [Spring Boot 프로파일 그룹](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles.groups)
- [Spring in Action, 5th Edition](https://www.manning.com/books/spring-in-action-fifth-edition) 