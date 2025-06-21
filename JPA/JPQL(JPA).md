## 자바 ORM 표준 JPA 프로그래밍 (김영한)
##  ◆ JPQL
JPQL은 Java Persistence Query Language의 약자로, JPA에서 사용하는 객체 지향 쿼리 언어이다.
SQL과 문법이 비슷하지만, 테이블이 아닌 엔티티 객체를 대상으로 쿼리한다는 점에서 큰 차이가 있다.

##  ▷ TypedQuery, Query
TypedQuery와 Query는 JPA에서 JPQL 쿼리를 실행할 때 사용하는 쿼리 실행 객체이다.
둘 다 EntityManager를 통해 생성하지만, 가장 큰 차이는 제네릭 타입 지정 여부에 있다.

### 📌 TypeQuery<T> (리턴 타입이 명확할 때)
```java
// Member는 엔티티 클래스
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> members = query.getResultList(); // 타입 안정성 높음
```
- ```Member.class```를 넘겨서, 반환 타입이 확실하기 때문에 컴파일 타임에 타입 체크가 가능하다.
- 캐스팅이 필요 없음.

### 📌 Query (타입이 명확하지 않을 때, 복합 조회 등)
```java
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
List<Object[]> results = query.getResultList();

for (Object[] row : results) {
    String username = (String) row[0];
    int age = (int) row[1];
}
```
- 여러 필드를 한 번에 조회하는 경우에는 ```Query```를 사용해야 한다.
- ```Object[]```형태로 받아야 하고, 형변환 필수.

## ▷ getResultList(), getSingleResult()
```query.getResultList()```와 ```query.getSingleResult()```는 
JPQL 실행 결과를 어떻게 받아올지를 결정하는 메서드이다.

### 📌 getResultList() (여러개의 결과를 조회할 때 사용)
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> members = query.getResultList();
```
- 결과가 없으면 빈 리스트 반환

### 📌 getSingleResult() (딱 하나의 결과만 예상될 때 사용)
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m WHERE m.id = :id", Member.class);
query.setParameter("id", 1L);
Member member = query.getSingleResult();
```
- 결과가 없을 때 NoResultException 발생.
- 결과가 2개 이상일 때 NonUniqueResultException 발생.

## ▷ 프로젝션
프로젝션은 SELECT 절에서 어떤 데이터를 추출할지 지정하는 것을 의미한다. 
다시 말해, 어떤 필드나 객체를 결과로 반환할지를 선택하는 것이다.

### ✅ 엔티티 전체를 조회하는 방식
```java
SELECT m FROM Member m
```
- ```Member```객체 전체를 조회한다.
- 반환 타입: ```List<Member>```

### ✅ 엔티티의 특정 필드만 조회하는 방식 (단일 값 or 여러 값)
```java
SELECT m.username FROM Member m
```
- 특정 필드 하나 조회 (단일 값)
- 반환 타입: ```List<String>```

```java
SELECT m.username, m.age FROM Member m
```
- 여러 필드 조회 (여러 값)
- 반환 타입: ```List<Object[]>```(배열 형태로 반환되기 때문에 인덱스로 접근 -> row[0], row[1])

### ✅ DTO로 조회하는 방식 (new 명령어 사용)
```java
SELECT new com.example.dto.MemberDto(m.username, m.age)
FROM Member m
```
- 생성자 방식으로 DTO 객체를 바로 생성해서 반환한다.
- DTO 클래스에 필드와 타입이 일치하는 생성자가 필요.
- 반환 타입: ```List<MemberDTO>```

## ▷ JOIN
JPQL에서는 조인을 할 때도 테이블 컬럼이 아니라 엔티티의 연관관계 필드를 기준으로 조인한다.

### ✨ 예시 코드:
```java
@Entity
public class Member {
    @ManyToOne
    private Team team;
}
```
위 관계가 있으면 JPQL에서는 다음처럼 조인을 할 수 있다.
```jpql
SELECT m FROM Member m JOIN m.team t
```

### 📌 내부 조인 (INNER JOIN)
```java
SELECT m
FROM Member m
JOIN m.team t
WHERE t.name = 'A'
```
→ ```Member```와 관련된 ```Team```중, 이름이 A인 팀만 결과에 포함

### 📌 외부 조인 (LEFT JOIN)
```java
SELECT m, t
FROM Member m
LEFT JOIN m.team t
```
→ 모든 ```Member```를 포함, 팀이 없으면 ```t```는```null```

### 📌 외부 조인 + ON 절
```java
SELECT m, t
FROM Member m
LEFT JOIN m.team t ON t.name = 'A'
```
→ 모든 ```Member```를 포함, 연관된 팀 중 이름이 ```A```인 경우만```t```에 조인됨.

### 📌 세타 조인 (연관관계 없음)
```java
SELECT m, t
FROM Member m
JOIN Team t ON m.username = t.name
```
→ ```Member```와 ```Team```사이에 연관관계가 없어도 임의 조건으로 조인 가능

