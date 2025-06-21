## 실용적인 테스트 가이드 (박우빈)
##  ◆ 스프링 레이어드 아키텍처
스프링에서 말하는 레이어드 아키텍처는 크게 4가지 계층으로 구성되는 아키텍처 스타일로서,
기본적으로 책임과 역할을 분리해서 시스템을 구조화하는 방식이다.
<br></br>

## ▷ 스프링의 레이어드 아키텍처 4계층

| 레이어 이름                    | 주요 역할               | 설명                                        |
|:--------------------------|:--------------------|:------------------------------------------|
| Presentation Layer (웹 계층) | 사용자 요청 처리, 응답 반환    | API를 받아들이고 응답하는 부분. 주로 Controller         |
| Service Layer (비즈니스 계층)   | 비즈니스 로직 처리          | 도메인 객체를 조합해 실제 요구사항을 수행                   |
| Repository Layer (영속성 계층) | 데이터베이스 접근           | DB나 외부 저장소와 연결, 주로 JpaRepository 같은 인터페이스 |
| Domain Layer (도메인 모델 계층)  | 비즈니스 객체 (Entity) 관리 | 핵심 비즈니스 규칙을 담은 객체(Entity)들                |
---

### 📌 Presentation Layer (프레젠테이션 계층)
- 사용자의 요청을 받는다. (HTTP 요청, WebSocket 요청 등)
- 요청을 가공해서 Service Layer로 전달한다.
- Service Layer로부터 받은 응답을 다시 사용자에게 돌려준다.
- 예외를 받아서 에러 응답을 만들기도 한다.

#### 주요 예시:
```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        OrderResponse response = orderService.createOrder(request);
        return ResponseEntity.ok(response);
    }
}
```
> Controller 는 얇게(Thin Controller) 유지하는 것이 권장된다.
> 복잡한 로직은 Service에 넘기는 것이 필요하다.
---
### 📌 Service Layer (서비스 계층)
- 비즈니스 규칙을 처리한다.
- 여러 도메인 객체나 Repository를 조합해서 실제 요구사항을 만족시킨다.
- 트랜잭션 관리(@Transactional)를 주로 여기서 한다.
- Presentation Layer가 어떤 방식으로 호출되든 (Web, Batch 등) Service Layer 로직은 동일하게 사용될 수 있다.

#### 주요 예시:
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final ProductService productService;

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {
        Product product = productService.findById(request.getProductId());
        Order order = new Order(product, request.getQuantity());
        orderRepository.save(order);
        return new OrderResponse(order);
    }
}
```
> Service는 "하나의 트랜잭션 단위" 로 묶이는 것이 일반적이다.
---
### 📌 Repository Layer (레포지토리 계층)
- DB와 직접 통신한다.
- CRUD 작업을 수행한다.
- 보통 Spring Data JPA의 JpaRepository를 상속하거나, 직접 @Repository를 선언해서 사용한다.

#### 주요 예시:
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```
#### 직접 커스텀 시: 
```java
@Repository
public class OrderRepositoryImpl {
    private final EntityManager em;

    public Order findById(Long id) {
        return em.find(Order.class, id);
    }
}
```
> Repository는 단순히 "저장/조회" 만 책임진다. 비즈니스 규칙을 이곳에 작성하면 안됨.
---
### 📌 Domain Layer (도메인 모델 계층)
- 비즈니스의 핵심 개념을 표현한다. (Entity, Value Object 등)
- 데이터 구조 + 관련 로직(메서드)을 가진다.
- 규칙이 복잡하다면, 도메인 객체 스스로 규칙을 가진 메서드를 만들어야 한다. (Rich Domain Model 지향)

#### 주요 예시:
```java
@Entity
public class Order {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Product product;

    private int quantity;

    protected Order() {}

    public Order(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    public int calculateTotalPrice() {
        return product.getPrice() * quantity;
    }
}
```
> Entity는 단순히 데이터만 담는 것이 아니라, 자신의 책임과 규칙을 스스로 수행하도록 설계하는 게 더 좋은 방식이다.

<br>

## ▷ 레이어 간 규칙
위에서 설명한 레이어 간 몇가지 지켜야할 규칙이 존재한다.

- Controller → Service → Repository 방향으로만 흐른다.
- 반대 방향으로 의존하지 않는다. (Repository가 Controller를 알면 안됨)
- 서로 다른 레이어를 침범하지 않는다. (Service가 HTTP 요청 객체를 직접 다루지 않는다 등)

> 요약하자면 레이어드 아키텍처는 책임과 역할을 분리해서 "유지보수" "확장" "테스트"를 쉽게 하기 위한 기본 아키텍처 스타일이다.





