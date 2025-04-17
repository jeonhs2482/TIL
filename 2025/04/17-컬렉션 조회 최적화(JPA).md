# 📚 2025.04.17 TIL
## 실전! 스프링 부트와 JPA 활용2 (김영한)
##  ◆ 컬렉션 조회 최적화
컬렉션인 일대다 관계(OneToMany)를 조회하고, 최적화 하는 방법을 알아보자.

## 1. 페치 조인 최적화
### ✨ 예시 코드:
```java
@GetMapping("/api/v3/orders")
public List<OrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithItem();
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());
    return result;
}
```
```java
public List<Order> findAllWithItem() {
    return em.createQuery(
        "select distinct o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d" +
                " join fetch o.orderItems oi" +
                " join fetch oi.item i", Order.class)
        .getResultList();
}
```
- 위 코드에서 페치 조인으로 인해 SQL은 1번만 실행된다.
- ```distinct```를 사용한 이유는 ```Order```와 ```OrderItem```은 일대다 관계이므로
조회 시 ```OrderItem```수 만큼 데이터베이스 row가 증가하게 된다. 그 결과 ```Order```엔티티의
조회 결과가 중복되어 증가하게되는데, JPA의 distinct는 SQL에 distinct를 추가하고,
중복되는 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다. (DB에서는 조회결과가 완전히 일치하지 않기 때문에
중복되는 조회결과를 모두 가져옴)
- 하지만 컬렉션을 페치 조인으로 조회하게되면 페이징이 불가능하다.
- 또한, 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다.

## 2. 페이징과 한계 돌파
위 1번 방법으로는 페이징이 불가능하다고 하였는데, 페이징 + 컬렉션 엔티티를 함께 조회하려면
아래 방법을 사용해야한다.

1. 먼저 ToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row 수를
증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
2. 컬렉션은 지연 로딩으로 조회한다.
3. 지연 로딩 성능 최적화를 위해 ```hibernate.default_batch_fetch_size```, ```@BatchSize```
를 적용한다.

### ✨ 예시 코드:
```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
        "select o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d", Order.class)
        .setFirstResult(offset)
        .setMaxResults(limit)
        .getResultList();
}
```
```java
@GetMapping("/api/v3.1/orders")
public List<OrderDto> ordersV3_page(@RequestParam(value = "offset", defaultValue = "0") int offset,
                                    @RequestParam(value = "limit", defaultValue = "100") int limit) {
    List<Order> orders = orderRepository.findAllWithMemberDelivery(offset,
            limit);
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(Collectors.toList());
    return result;
}
```
### 최적화 옵션:
```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000\
```
- 위 방법으로 컬렉션 조회 시 쿼리 호출 수가 1 + N + M 에서 1 + 1 + 1로 최적화 된다.
- 페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB데이터 전송량이 감소한다.
  (페치 조인 방식은 증가된 row의 데이터를 모두 전송하기 때문)
- 컬렉션 페치 조인 방식은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.
- 결론적으로 ToOne관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서, ToOne 관계는
페치조인으로 쿼리 수를 줄이고, 나머지는 ```hibernate.default_batch_fetch_size```로
최적화 하는 방식으로 해결하자.