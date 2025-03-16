---
layout: post
title: 스프링 부트에서 Entity 설계 및 구현 가이드
date: 2025-03-11
categories: [Spring Boot]
tags: [spring, spring boot, jpa, entity, 엔티티, 데이터 모델링, hibernate]
---

이전 포스팅에서는 DTO 패턴의 설계 및 구현에 대해 알아보았습니다. 이번에는 스프링 부트 애플리케이션에서 JPA Entity를 효과적으로 설계하고 구현하는 방법에 대해 자세히 알아보겠습니다. Entity는 데이터베이스 테이블과 매핑되는 객체로, 애플리케이션의 데이터 모델을 표현하는 핵심 요소입니다.

## Entity란 무엇인가?

Entity는 JPA(Java Persistence API)에서 데이터베이스 테이블과 매핑되는 자바 객체입니다. 각 Entity 인스턴스는 데이터베이스 테이블의 한 행(row)에 해당합니다. Entity는 다음과 같은 특징을 가집니다:

1. **영속성(Persistence)**: 데이터베이스에 저장되고 관리됨
2. **식별성(Identity)**: 고유한 식별자(ID)를 가짐
3. **상태 변화 추적**: JPA가 Entity의 상태 변화를 감지하고 데이터베이스에 반영
4. **관계 표현**: 다른 Entity와의 관계(일대일, 일대다, 다대다 등)를 표현

## Entity 설계 기본 원칙

### 1. 기본 Entity 구조

가장 기본적인 Entity 클래스는 다음과 같은 구조를 가집니다:

```java
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false)
    private String name;
    
    @Column(name = "price")
    private Double price;
    
    @Column(name = "description", length = 1000)
    private String description;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // 기본 생성자 (JPA 요구사항)
    public Product() {
    }
    
    // 생성자, getter, setter 등
}
```

### 2. 식별자(ID) 설계

Entity의 식별자는 다양한 방식으로 생성할 수 있습니다:

```java
// 자동 증가(Auto Increment)
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// 시퀀스 사용
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "product_seq")
@SequenceGenerator(name = "product_seq", sequenceName = "product_sequence", allocationSize = 1)
private Long id;

// UUID 사용
@Id
@GeneratedValue(generator = "uuid2")
@GenericGenerator(name = "uuid2", strategy = "uuid2")
@Column(columnDefinition = "VARCHAR(36)")
private String id;

// 복합 키 사용
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    
    // equals, hashCode 구현 필수
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
    
    // 다른 필드들...
}
```

### 3. 관계 매핑

Entity 간의 관계는 다음과 같이 매핑할 수 있습니다:

#### 일대다(One-to-Many) 관계

```java
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    // 편의 메서드
    public void addOrder(Order order) {
        orders.add(order);
        order.setCustomer(this);
    }
    
    public void removeOrder(Order order) {
        orders.remove(order);
        order.setCustomer(null);
    }
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    // 다른 필드들...
}
```

#### 다대다(Many-to-Many) 관계

```java
// 중간 테이블 없이 직접 매핑 (권장하지 않음)
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}

// 중간 Entity를 사용한 매핑 (권장)
@Entity
public class StudentCourse {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;
    
    @ManyToOne
    @JoinColumn(name = "course_id")
    private Course course;
    
    private LocalDateTime enrolledAt;
    
    // 추가 필드 및 메서드
}
```

## Entity 설계 고급 기법

### 1. 상속 관계 매핑

JPA에서는 상속 관계를 다양한 전략으로 매핑할 수 있습니다:

#### 단일 테이블 전략(SINGLE_TABLE)

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type")
public abstract class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private BigDecimal amount;
    
    // 공통 필드 및 메서드
}

@Entity
@DiscriminatorValue("CREDIT_CARD")
public class CreditCardPayment extends Payment {
    private String cardNumber;
    private String cardHolderName;
    private LocalDate expiryDate;
    
    // 신용카드 관련 필드 및 메서드
}

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPayment extends Payment {
    private String bankName;
    private String accountNumber;
    
    // 계좌이체 관련 필드 및 메서드
}
```

#### 조인 전략(JOINED)

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String manufacturer;
    private String model;
    
    // 공통 필드 및 메서드
}

@Entity
public class Car extends Vehicle {
    private int numberOfDoors;
    private String fuelType;
    
    // 자동차 관련 필드 및 메서드
}

@Entity
public class Motorcycle extends Vehicle {
    private int engineCapacity;
    private boolean hasSideCar;
    
    // 오토바이 관련 필드 및 메서드
}
```

### 2. 임베디드 타입(Embedded Type)

값 타입을 사용하여 Entity를 더 구조화할 수 있습니다:

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    private String country;
    
    // 생성자, getter, setter 등
}

@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @Embedded
    private Address address;
    
    // 다른 필드 및 메서드
}

@Entity
public class Company {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "office_street")),
        @AttributeOverride(name = "city", column = @Column(name = "office_city")),
        @AttributeOverride(name = "state", column = @Column(name = "office_state")),
        @AttributeOverride(name = "zipCode", column = @Column(name = "office_zip_code")),
        @AttributeOverride(name = "country", column = @Column(name = "office_country"))
    })
    private Address officeAddress;
    
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "shipping_street")),
        @AttributeOverride(name = "city", column = @Column(name = "shipping_city")),
        @AttributeOverride(name = "state", column = @Column(name = "shipping_state")),
        @AttributeOverride(name = "zipCode", column = @Column(name = "shipping_zip_code")),
        @AttributeOverride(name = "country", column = @Column(name = "shipping_country"))
    })
    private Address shippingAddress;
    
    // 다른 필드 및 메서드
}
```

### 3. 생명주기 콜백(Lifecycle Callbacks)

Entity의 생명주기 이벤트를 처리하기 위한 콜백 메서드를 정의할 수 있습니다:

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private Double price;
    
    @Column(updatable = false)
    private LocalDateTime createdAt;
    
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    // 다른 필드 및 메서드
}
```

## Entity 설계 모범 사례

### 1. 양방향 관계 관리

양방향 관계에서는 연관관계의 주인(owner)을 명확히 하고, 편의 메서드를 통해 일관성을 유지하세요:

```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private String content;
    
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
    
    // 편의 메서드
    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }
    
    public void removeComment(Comment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }
    
    // 다른 필드 및 메서드
}

@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;
    
    // 다른 필드 및 메서드
}
```

### 2. 지연 로딩(Lazy Loading) 활용

성능 최적화를 위해 연관 관계에서 지연 로딩을 적절히 활용하세요:

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩
    @JoinColumn(name = "customer_id")
    private Customer customer;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> orderItems = new ArrayList<>();
    
    // 다른 필드 및 메서드
}
```

### 3. 도메인 로직 포함

Entity에 관련 도메인 로직을 포함시켜 응집도를 높이세요:

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> orderItems = new ArrayList<>();
    
    private LocalDateTime orderedAt;
    
    // 도메인 로직
    public BigDecimal calculateTotalAmount() {
        return orderItems.stream()
                .map(OrderItem::getSubtotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    public void markAsShipped() {
        if (status != OrderStatus.PAID) {
            throw new IllegalStateException("Order must be paid before shipping");
        }
        status = OrderStatus.SHIPPED;
    }
    
    public void cancel() {
        if (status == OrderStatus.SHIPPED || status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Cannot cancel shipped or delivered order");
        }
        status = OrderStatus.CANCELLED;
    }
    
    // 다른 필드 및 메서드
}

@Entity
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id")
    private Product product;
    
    private int quantity;
    private BigDecimal price;
    
    // 도메인 로직
    public BigDecimal getSubtotal() {
        return price.multiply(new BigDecimal(quantity));
    }
    
    // 다른 필드 및 메서드
}
```

### 4. 낙관적 락(Optimistic Locking) 활용

동시성 제어를 위해 낙관적 락을 활용하세요:

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal price;
    private int stockQuantity;
    
    @Version
    private Long version;
    
    // 도메인 로직
    public void decreaseStock(int quantity) {
        if (stockQuantity < quantity) {
            throw new IllegalArgumentException("Not enough stock available");
        }
        stockQuantity -= quantity;
    }
    
    // 다른 필드 및 메서드
}
```

## 결론

Entity 설계는 데이터베이스 스키마와 객체 지향 모델 사이의 균형을 맞추는 중요한 작업입니다. 적절한 관계 매핑, 상속 전략, 임베디드 타입 활용, 그리고 도메인 로직 포함을 통해 더 견고하고 유지보수하기 쉬운 데이터 모델을 구축할 수 있습니다.

JPA의 다양한 기능을 활용하여 객체 지향적인 설계를 유지하면서도 효율적인 데이터베이스 접근이 가능한 Entity를 설계하세요. 특히 양방향 관계 관리, 지연 로딩 활용, 그리고 낙관적 락 등의 모범 사례를 따르면 성능과 유지보수성을 모두 확보할 수 있습니다.

다음 포스팅에서는 스프링 데이터 JPA를 활용한 Repository 설계 및 구현 방법에 대해 알아보도록 하겠습니다.

## 참고 자료
- [Spring Data JPA 공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Hibernate 공식 문서](https://hibernate.org/orm/documentation/)
- [JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)
- [Baeldung - JPA Entity 관계 매핑](https://www.baeldung.com/jpa-entity-relationships)
- [Baeldung - JPA 상속 전략](https://www.baeldung.com/hibernate-inheritance)
- [Vlad Mihalcea의 하이버네이트 블로그](https://vladmihalcea.com/tutorials/hibernate/) 