# 📚 2025.04.14 TIL
## 자바 ORM 표준 JPA 프로그래밍 (김영한)
##  ◆ 페치 조인(fetch join)
페치 조인은 연관된 엔티티를 SQL 조인으로 함께 조회하고, 그 결과를 엔티티에 매핑하는
JPQL의 기능이다. N + 1 문제 해결에 자주 사용된다.

## 1. N + 1 문제란?
#### 엔티티 구조 예시:
```java
@Entity
public class Member {
    @Id
    private Long id;
    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}

@Entity
public class Team {
    @Id
    private Long id;
    private String name;
}
```
### N + 1 문제의 개념
- ```Member```엔티티 리스트를 1번의 쿼리로 조회 (1번)
- 각 ```Member```가 가진 ```Team```정보를 조회하기 위해 추가로 N번의 쿼리가
실행됨 (N번)
- 결과적으로 총 N + 1번의 쿼리가 실행됨.

### ✨ 예시 코드 (LAZY 로딩 + for 루프)
```java
List<Member> members = em.createQuery("select m from Member m", Member.class)
                         .getResultList();

for (Member member : members) {
    System.out.println(member.getTeam().getName()); // .getName() 호출시  Team 조회
}
```
### ▶ 실행되는 쿼리 (예: Member가 3명일 때)
```sql
-- 1번 실행
select * from Member;

-- 3번 실행 (Team이 LAZY이므로 각 member.team 접근 시 마다 조회)
select * from Team where id = ?;
select * from Team where id = ?;
select * from Team where id = ?;
```

### ✨ 예시 코드 (EAGER 로딩)
```java
List<Member> members = em.createQuery("select m from Member m", Member.class)
        .getResultList();
```

### ▶ 실행되는 쿼리 (Hibernate는 즉시 로딩 시에도 기본적으로 조인 없이 실행함)
```sql
-- 1번 실행
select * from Member;

-- 3번 실행 (각 Team을 별도로 조회)
select * from Team where id = ?;
select * from Team where id = ?;
select * from Team where id = ?;
```
- EAGER 로딩, LAZY 로딩 모두 N + 1 문제가 발생할 수 있음.
- 이를 해결하기 위해 fetch join을 사용해야함.

## 2. join vs join fetch의 차이

### ✅ join
```jpql
select m from Member m join m.team
```
- SQL 상으로는 내부 조인이 수행돼서 ```Member```와 ```Team```을 같이 조회하는 것처럼 보임.
- 하지만 JPA는 여전히 ```Member```만 영속성 컨텍스트에 채워놓고, ```Team```은 프록시 객체로 유지함 (LAZY이기 때문)
- 따라서 ```member.getTeam().getName()```을 호출하면 또 쿼리 발생

### ✅ join fetch
```jpql
select m from Member m join fetch m.team
```
- SQL은 위와 동일하게 조인 수행
- 그러나 JPA는 이 쿼리 결과를 보고 ```Member```와 ```Team```을 함께 영속성 컨텍스트에 로딩
- ```Team```도 실제 객체로 채워지기 때문에 추가 쿼리 없음

## 3. 페치 조인의 한계
- 페치 조인 대상에는 별칭을 줄 수 없다.
  - 하이버네이트는 가능하지만 가급적 사용하지 않는 것이 좋다.
- 둘 이상의 컬렉션은 페치 조인 할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
  - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능.
  - 하이버네이트는 경고 로그를 남기고 메모리에서 페이징. (매우 위험)
️