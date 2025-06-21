## 자바 ORM 표준 JPA 프로그래밍 (김영한)
##  ◆ 프록시와 연관관계 관리
JPA 에서 프록시 객체는 지연 로딩을 구현하기 위한 핵심 메커니즘이다.

## ▷ 프록시 객체란?
- 프록시 객체는 실제 엔티티 객체를 대신하는 가짜 객체이다.
- JPA는 엔티티를 실제로 DB에서 조회하지 않고도 일단 프록시 객체를 생성해서
  반환할 수 있고 이 프록시 객체는 필요한 시점에 실제 데이터를 DB에서 조회한다.

### ✨ 예시 코드:
```java
@Entity
public class Order {
    @Id
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}
```
### 내부 동작:
1. order.getMember()를 호출한다.
2. JPA는 Hibernate 프록시 객체를 Member 대신 생성해 놓는다.
   (Member$HibernateProxy 같은 클래스)
3. 프록시 객체는 내부적으로 실제 Member의 ID를 가지고 있음.
4. member.getName()처럼 필드를 접근하려는 순간, 그제서야 DB를
   조회해서 진짜 Member 객체를 채워 넣는다(초기화).

### ✨ 간단한 프록시 예제:
```java
Order order = em.find(Order.class, 1L); // order 조회
Member memberProxy = order.getMember(); // 프록시 객체 반환 (아직 SELECT 안함)

System.out.println("class: " + memberProxy.getClass()); 
// com.example.Member$HibernateProxy

System.out.println("name: " + memberProxy.getName()); 
// 이 시점에 SELECT 쿼리 발생

System.out.println("email: " + memberProxy.getEmail()); 
// SELECT 없음, 이미 초기화됨
// 프록시 객체는 한 번 초기화되면 그 이후에는 다시 DB를 조회하지 않음.
```

## ▷ 프록시의 특징
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님,
  초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함
  (== 비교시 실패, 대신 instance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를
  호출해도 실제 엔티티를 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면
  문제 발생 (하이버 네이트는 org.hibernate.LazyInitializationException)
  예외를 터뜨림

## ▷ getReference() 호출 시 무슨 일이 벌어지나?
```java
Member proxy = em.getReference(Member.class, 1L);
```
1. 영속성 컨텍스트(1차 캐시)를 먼저 확인
- Member 엔티티의 ID가 이미 캐시에 존재하면 실제 엔티티를 반환 (find()와 동일한 동작).
2. 존재하지 않으면
- Hibernate는 해당 ID에 대한 프록시 객체를 생성
- 이 프록시 객체를 영속성 컨텍스트에 등록
3. 이후 동일한 엔티티 ID로 find()를 호출하든, getReference()를 호출하든
   항상 같은 인스턴스(1차 캐시)를 반환한다. </br></br>

## ◆ 영속성 전이: CASCADE
특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속상태로 만들고
싶을 때 사용. 엔티티 상태 변경을 연관된 엔티티까지 전파해주는 옵션이다.

### ✨ 사용 예시:
```java
@Entity
public class Parent {
  @Id @GeneratedValue
  private Long id;

  private String name;

  @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
  private List<Child> children = new ArrayList<>();

  public void addChild(Child child) {
    children.add(child);
    child.setParent(this);
  }
}

@Entity
public class Child {
  @Id @GeneratedValue
  private Long id;

  private String name;

  @ManyToOne
  private Parent parent;
}
```
```java
Parent parent = new Parent();
parent.setName("엄마");

Child child1 = new Child();
child1.setName("자식1");

Child child2 = new Child();
child2.setName("자식2");

parent.addChild(child1);
parent.addChild(child2);

em.persist(parent); // 부모만 저장하면, 자식들도 자동 저장됨!
```
### 위 예제 코드에서 CascadeType.ALL을 사용한 덕분에:
- em.persist(child1) / em.persist(child2)를 하지 않아도됨.
- 삭제 시에도 em.remove(parent)만 하면 자식도 같이 삭제됨. </br></br>

## ◆ 고아 객체 삭제
orphanRemoval = true 설정 시 부모 엔티티와의 연관관계가 끊긴
자식 엔티티(고아 객체)를 자동으로 삭제해준다. 연관 컬렉션에서 자식을
제거하거나 관계를 끊으면 DB에서도 DELETE 쿼리가 자동 발생한다.

### ✨ 사용 예시:
```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }

    public void removeChild(Child child) {
        children.remove(child);
        child.setParent(null);
    }
}

@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Parent parent;
}
```
```java
Parent parent = em.find(Parent.class, 1L);
Child child = parent.getChildren().get(0);

parent.removeChild(child);  // 관계 끊기
// 트랜잭션 커밋 시 -> 자동으로 DELETE FROM child where id=?
```
```orphanRemoval = true``` 없으면: 자식은 연관만 끊기고 DB에는 그대로 남음. </br>
```orphanRemoval = true``` 있으면: 연관 끊기면 자식도 자동 삭제됨.