# 백엔드 아키텍처 완전 정리

> Service, Domain, POJO, DTO, Entity의 역할과 전체 데이터 흐름 이해하기

[![Tech](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=spring-boot&logoColor=white)](https://spring.io/)
[![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)](https://www.java.com/)
[![Architecture](https://img.shields.io/badge/Architecture-Clean-blue?style=flat-square)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

## 📑 목차

- [1. Service의 진짜 역할](#1-service의-진짜-역할)
- [2. Service에 비즈니스 로직이 있을 때의 문제점](#2-service에-비즈니스-로직이-있을-때의-문제점)
- [3. Domain - 비즈니스 로직의 주인](#3-domain---비즈니스-로직의-주인)
- [4. POJO (Plain Old Java Object)](#4-pojo-plain-old-java-object)
- [5. 전체 흐름: 요청부터 응답까지](#5-전체-흐름-요청부터-응답까지)
- [6. 전체 아키텍처 다이어그램](#6-전체-아키텍처-다이어그램)
- [7. 계층별 역할 정리](#7-계층별-역할-정리)
- [8. 주요 변환 과정](#8-주요-변환-과정)
- [9. 언제 Domain을 사용하는가](#9-언제-domain을-사용하는가)
- [10. 핵심 원칙 요약](#10-핵심-원칙-요약)

---

## 1. Service의 진짜 역할

### 💡 핵심 개념

**Service는 비즈니스 로직을 담당하지 않는다. 비즈니스 로직의 실행(조율)만을 담당한다.**

### Service가 하는 일 ✅

- 여러 Domain 객체를 조합
- Repository를 통한 데이터 조회/저장
- 트랜잭션 관리
- DTO ↔ Domain/Entity 변환
- 외부 서비스 호출 (이메일, 알림 등)

### Service가 하지 말아야 할 일 ❌

- 복잡한 계산 로직
- 비즈니스 규칙 (할인율, 검증 규칙 등)
- if문이 많은 분기 처리

---

## 2. Service에 비즈니스 로직이 있을 때의 문제점

### ❌ 문제 1: 코드 중복

같은 로직이 여러 Service에 복붙됨
- `OrderService`, `CartService`, `RefundService`에 같은 할인 계산 로직

### ❌ 문제 2: 유지보수의 어려움

- 비즈니스 규칙 변경 시 모든 Service를 찾아서 수정해야 함
- 하나라도 빠뜨리면 버그 발생

### ❌ 문제 3: 데이터와 로직의 분리로 인한 위험성

```java
// 필요한 데이터를 매번 파라미터로 전달
private int calculateDiscount(Customer customer, int basePrice) { ... }

// 잘못된 customer를 넘기는 실수 가능 😱
int discount = calculateDiscount(wrongCustomer, price);
```

### ❌ 문제 4: 책임 소재의 불명확

- "할인 계산은 어느 Service에 있어야 하지?"
- 여러 곳에 흩어져서 찾기 어려움

---

## 3. Domain - 비즈니스 로직의 주인

### 💡 핵심 개념

**Domain은 비즈니스 로직을 여러 메서드로 분리하여 관리한다.**

### ✨ Domain의 장점

#### 📖 가독성 향상

Service의 100줄 메서드 → Domain의 10줄 메서드 10개로 분리

```java
// ❌ Service (복잡)
int result = ... // 100줄의 계산 로직

// ✅ Domain (깔끔)
Money total = order.calculateTotalPrice();
Money discount = customer.calculateDiscount(total);
Money shipping = order.calculateShippingFee();
```

#### ♻️ 재사용성

- 한 번 작성한 로직을 여러 Service에서 재사용
- 코드 중복 제거

#### 🔧 유지보수성

- 비즈니스 규칙 변경 시 Domain만 수정
- 모든 Service에 자동 반영

#### 🎯 책임의 명확화

- `Customer`의 로직은 `Customer` Domain에
- `Order`의 로직은 `Order` Domain에
- 어디를 수정해야 할지 명확함

---

## 4. POJO (Plain Old Java Object)

### 📌 정의

**Plain Old Java Object = 어떠한 프레임워크나 라이브러리에도 의존하지 않는 순수한 자바 객체**

### 특징

- ✅ JPA, Spring 어노테이션 없음
- ✅ 특정 클래스 상속 없음
- ✅ 특정 인터페이스 구현 없음
- ✅ 순수 자바 코드만 사용

### 🔄 POJO의 역할

#### 비즈니스 로직 실행 흐름

```
Entity (DB) → POJO Domain (비즈니스 로직) → Entity (DB)
```

1. Entity에서 필요한 정보를 추출하여 POJO Domain 생성
2. POJO로 비즈니스 로직 실행
3. 실행 결과를 다시 Entity로 변환

### 🎁 POJO의 장점

#### 🔌 기술 독립성

- JPA가 바뀌어도 POJO는 변경 없음
- Spring이 바뀌어도 POJO는 변경 없음
- DB 구조가 바뀌어도 비즈니스 로직은 안전

#### 🧪 테스트 용이성

- DB 없이 즉시 테스트 가능
- Spring 컨테이너 없이 테스트 가능
- 빠른 단위 테스트 작성

#### 🛡️ 안정성

- 의존성이 없어 외부 변경에 영향을 받지 않음
- 순수 자바 로직이므로 예측 가능한 동작

---

## 5. 전체 흐름: 요청부터 응답까지

### 🔄 7단계 프로세스

### 📝 상세 단계

#### 1️⃣ 클라이언트 요청

```javascript
// JavaScript 객체 생성
const orderData = { productId: 10, quantity: 2 };

// JSON 문자열로 변환 및 전송
fetch('/api/orders', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(orderData)
});
```

#### 2️⃣ Spring 서버 수신 및 DTO 변환

```java
@PostMapping("/api/orders")
public OrderResponseDTO createOrder(
    @RequestHeader("Authorization") String token,
    @RequestBody OrderRequestDTO dto  // Jackson이 JSON → DTO 자동 변환
) {
    return orderService.createOrder(token, dto);
}
```

#### 3️⃣ 인증 및 Entity 조회

```java
// 토큰 검증 및 사용자 ID 추출
Long userId = tokenProvider.getUserIdFromToken(token);

// DB에서 Entity 조회
UserEntity user = userRepository.findById(userId).orElseThrow();
ProductEntity product = productRepository.findById(dto.getProductId()).orElseThrow();
```

#### 4️⃣ 비즈니스 로직 실행

```java
// Entity → POJO Domain 변환
User userDomain = user.toDomain();
Product productDomain = product.toDomain();

// POJO Domain으로 비즈니스 로직 실행
Order order = Order.create(userDomain, productDomain, dto.getQuantity());
Money totalPrice = order.calculateTotalPrice();
Money discount = order.calculateDiscount();

// POJO Domain → Entity 변환 및 저장
OrderEntity orderEntity = OrderEntity.from(order);
orderRepository.save(orderEntity);
```

#### 5️⃣ 응답 생성

```java
// Entity → ResponseDTO 변환 (필요한 정보만 선택)
return OrderResponseDTO.builder()
    .orderId(orderEntity.getId())
    .totalPrice(orderEntity.getTotalPrice())
    .status(orderEntity.getStatus())
    .build();
// Jackson이 ResponseDTO → JSON 자동 변환
```

#### 6️⃣ 클라이언트 수신

```javascript
// JSON 문자열 수신
const response = await fetch('/api/orders', { ... });

// JSON → JavaScript 객체 변환
const data = await response.json();
// { orderId: 123, totalPrice: 50000, status: "PENDING" }
```

#### 7️⃣ 화면 갱신

```javascript
// DOM 업데이트
document.getElementById('orderId').textContent = data.orderId;
document.getElementById('price').textContent = data.totalPrice;
```

---

## 6. 전체 아키텍처 다이어그램

```
┌─────────────────┐                    ┌──────────────────┐                    ┌──────────┐
│   클라이언트      │                    │   백엔드 서버       │                    │ 데이터베이스 │
│                 │                    │                  │                    │          │
│ JavaScript 객체  │                    │                  │                    │          │
│       ↓         │                    │                  │                    │          │
│  JSON 문자열     │ ────HTTP 전송────→  │  JSON 문자열      │                    │          │
│                 │                    │       ↓          │                    │          │
│                 │                    │  Jackson 변환     │                    │          │
│                 │                    │       ↓          │                    │          │
│                 │                    │  RequestDTO      │                    │          │
│                 │                    │       ↓          │                    │          │
│                 │                    │  Controller      │                    │          │
│                 │                    │       ↓          │                    │          │
│                 │                    │  Service ────────┼──→ Repository ───→ │   DB     │
│                 │                    │       ↓          │                    │          │
│                 │                    │  Entity ←────────┼─── Repository ←─── │   DB     │
│                 │                    │       ↓          │                    │          │
│                 │                    │  POJO Domain     │                    │          │
│                 │                    │  (비즈니스 로직)   │                    │          │
│                 │                    │       ↓          │                    │          │
│                 │                    │  Entity ─────────┼──→ Repository ───→ │   DB     │
│                 │                    │       ↓          │                    │          │
│                 │                    │  ResponseDTO     │                    │          │
│                 │                    │       ↓          │                    │          │
│  JSON 문자열     │ ←───HTTP 응답────  │  JSON 문자열      │                    │          │
│       ↓         │                    │                  │                    │          │
│ JavaScript 객체  │                    │                  │                    │          │
│       ↓         │                    │                  │                    │          │
│   화면 갱신      │                    │                  │                    │          │
└─────────────────┘                    └──────────────────┘                    └──────────┘
```

---

## 7. 계층별 역할 정리

### 🎮 Controller

```java
@RestController
public class OrderController {
    @PostMapping("/api/orders")
    public OrderResponseDTO createOrder(@RequestBody OrderRequestDTO dto) {
        return orderService.createOrder(dto);
    }
}
```

**역할:**
- HTTP 요청/응답 처리
- RequestDTO 수신 (JSON → DTO 자동 변환)
- ResponseDTO 반환 (DTO → JSON 자동 변환)
- 토큰 등 헤더 정보 처리

---

### 🎯 Service

```java
@Service
@Transactional
public class OrderService {
    public OrderResponseDTO createOrder(OrderRequestDTO dto) {
        // 조율자 역할
        User user = userRepository.findById(dto.getUserId());
        Product product = productRepository.findById(dto.getProductId());
        
        Order order = Order.create(user, product);  // Domain 호출
        orderRepository.save(order);
        
        return OrderResponseDTO.from(order);
    }
}
```

**역할:** (조율자)
- 토큰 검증 및 사용자 인증
- Repository를 통한 Entity 조회
- Entity ↔ POJO Domain 변환
- 여러 Domain 객체 조합
- 트랜잭션 관리
- 외부 서비스 호출

---

### 🧠 Domain (POJO)

```java
public class Order {  // 어노테이션 없음!
    private Customer customer;
    private Money totalPrice;
    
    // 비즈니스 로직
    public Money calculateDiscount() {
        return customer.calculateDiscountFor(this);
    }
    
    public Money getFinalPrice() {
        return totalPrice.subtract(calculateDiscount());
    }
}
```

**역할:** (비즈니스 로직의 주인)
- 계산, 검증, 변환 등 모든 비즈니스 규칙
- 기술에 독립적 (JPA, Spring 무관)
- 테스트 용이

---

### 💾 Repository

```java
public interface OrderRepository extends JpaRepository<OrderEntity, Long> {
    Optional<OrderEntity> findById(Long id);
    List<OrderEntity> findByUserId(Long userId);
}
```

**역할:**
- DB CRUD 작업
- Entity 조회/저장
- 쿼리 실행

---

### 🗄️ Entity

```java
@Entity
@Table(name = "orders")
public class OrderEntity {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column
    private Integer totalPrice;
    
    // Entity → Domain 변환
    public Order toDomain() {
        return Order.builder()
            .totalPrice(Money.of(this.totalPrice))
            .build();
    }
}
```

**역할:**
- DB 테이블과 1:1 매핑
- JPA 어노테이션 포함
- 영속성 컨텍스트 관리

---

### 📦 DTO

```java
// RequestDTO - 클라이언트 → 서버
public class OrderRequestDTO {
    private Long productId;
    private Integer quantity;
}

// ResponseDTO - 서버 → 클라이언트
public class OrderResponseDTO {
    private Long orderId;
    private Integer totalPrice;
    private String status;
    // password, 내부 정보는 없음!
}
```

**역할:**
- 클라이언트 ↔ 서버 데이터 전송
- 필요한 정보만 선택적 노출
- 민감정보 숨김

---

## 8. 주요 변환 과정

### 🔄 JSON ↔ DTO (자동 변환)

```
[요청 시]
JSON 문자열 (클라이언트) → Jackson (Spring) → RequestDTO (서버)

[응답 시]
ResponseDTO (서버) → Jackson (Spring) → JSON 문자열 (클라이언트)
```

**변환 주체:** Spring의 Jackson 라이브러리 (자동)

---

### 🔄 Entity ↔ POJO Domain (수동 변환)

```java
// Entity → POJO
UserEntity entity = userRepository.findById(id);
User user = entity.toDomain();  // 수동 변환

// POJO → Entity
User user = ... // 비즈니스 로직 실행
UserEntity entity = UserEntity.from(user);  // 수동 변환
```

**변환 주체:** 개발자가 직접 작성 (수동)

---

## 9. 언제 Domain을 사용하는가?

### ✅ Domain이 필요한 경우

- 복잡한 계산 로직 (가격, 할인, 수수료 등)
- 여러 조건의 분기 처리
- 동일한 로직을 여러 Service에서 사용
- 비즈니스 규칙이 자주 변경됨

### ❌ Domain이 불필요한 경우

- 단순 CRUD (조회, 생성, 수정, 삭제만)
- 비즈니스 로직이 거의 없음
- 소규모 프로젝트

### 💡 실무 접근법

```
1. 처음에는 Service에 로직 작성
2. 복잡해지면 Domain으로 분리
3. 점진적 리팩토링
```

---

## 10. 핵심 원칙 요약

### 🎯 관심사의 분리

| 계층 | 책임 |
|------|------|
| Controller | HTTP 처리 |
| Service | 조율 및 실행 |
| Domain | 비즈니스 로직 |
| Repository | 데이터 접근 |
| Entity | DB 매핑 |
| DTO | 데이터 전송 |

### 📐 단일 책임 원칙

- 각 계층은 하나의 책임만
- 변경 이유가 하나만 있어야 함

### ➡️ 의존성 방향

```
Controller → Service → Domain
                    ↓
                Repository → Entity
```

### 🔒 보안 원칙

- Entity는 서버 내부에만 존재
- 클라이언트는 DTO만 접근
- 민감정보는 ResponseDTO에서 제외

---

## 📚 참고 자료

### 추천 학습 자료

- **Spring Boot 공식 문서**: https://spring.io/projects/spring-boot
- **도메인 주도 설계 철저 입문** (성재욱 저)
- **Clean Architecture** (Robert C. Martin)

### 관련 패턴

- Strategy Pattern (할인 계산 등)
- Factory Pattern (객체 생성)
- Builder Pattern (DTO 생성)

---

## 🎯 핵심 기억 사항

```
✅ Service는 조율자, Domain이 비즈니스 로직의 주인
✅ POJO로 기술 독립적인 비즈니스 로직 구현
✅ DTO는 양방향 데이터 전송의 통역사
✅ Jackson이 JSON ↔ DTO 자동 변환
✅ 간단하면 Service로, 복잡하면 Domain 추가
```

---

## 🚀 실무 적용

- 프로젝트 규모에 맞게 선택
- 처음부터 완벽하게 만들려 하지 말 것
- 필요에 따라 점진적으로 개선

---

## 📝 License

이 문서는 학습 목적으로 자유롭게 사용 가능합니다.

---

## 👤 Author

**Backend Architecture Study**

- 작성일: 2024-12-23
- 버전: 1.0
- 대상: 백엔드 개발 학습자

---

<div align="center">

</div>
