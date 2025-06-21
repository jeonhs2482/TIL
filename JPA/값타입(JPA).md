## 자바 ORM 표준 JPA 프로그래밍 (김영한)
##  ◆ 값 타입
JPA에서 값 타입은 크게 세가지 타입으로 분류된다.
- 기본값 타입 
  - 자바 기본 타입(int, double)
  - 래퍼 클래스(Integer, Long)
  - String
- 임베디드 타입(복합 값 타입)
- 컬렉션 값 타입

## ▷ 임베디드 타입
임베디드 타입은 값타입을 엔티티에 포함시켜서 재사용 가능한 구조를 만들 때 사용된다.
쉽게 말해, 하나의 객체를 엔티티 안에 포함시키는 개념이다.

### ✨ 예시 코드:
```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
```
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @Embedded
    private Address address;
}
```
- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.

### ✅ 임베디드 타입 사용 시 장점:
- 재사용성: 같은 구조(예: 주소)를 여러 엔티티에서 재사용 가능.
- 객체지향적 설계: 의미 있는 값 객체를 만들 수 있음.
- 명확한 설계: 엔티티의 일부 필드를 묶어서 의미 있는 단위로 표현 가능.

### ❗임베디드 타입 사용 시 주의할 점
- 불변 객체로 설계해야함. (Setter 없이, 생성자로만 초기화)
- 엔티티의 생명주기를 따라가기 때문에, 공유되면 안된다. (Address를 두 회원이 공유 x)

### 임베디드 타입 내부에 유틸성 메서드 추가
임베디드 타입 내부에 유틸성 메서드를 추가하여 단순히 데이터를 담는 그릇이 아니라, 
도메인 로직을 캡슐화한 객체로 활용할 수 있다.

### ✨ 예시 코드:
```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;

    // 생성자
    protected Address() {} // JPA 기본 생성자
    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    // 유틸 메서드: 주소가 같은지 비교
    public boolean isSameCity(Address other) {
        return this.city != null && this.city.equals(other.city);
    }

    @Override
    public String toString() {
        return city + " " + street + " " + zipcode;
    }
}
```
위 예시처럼 임베디드 타입 내부에 메서드를 추가하여 더 응집력 있는 설계를 할 수 있다.

### 컬럼 이름 충돌 방지
하나의 엔티티에 두 개의 @Embedded 필드를 가지는 경우, 컬럼명이 중복될 수 있는데,
이때는 @AttributeOverrides로 해결한다.

### ✨ 예시 코드:
```java
@Embedded
@AttributeOverrides({
        @AttributeOverride(name="city", column=@Column(name="home_city")),
        @AttributeOverride(name="street", column=@Column(name="home_street")),
        @AttributeOverride(name="zipcode", column=@Column(name="home_zipcode"))
})
private Address homeAddress;

@Embedded
@AttributeOverrides({
        @AttributeOverride(name="city", column=@Column(name="work_city")),
        @AttributeOverride(name="street", column=@Column(name="work_street")),
        @AttributeOverride(name="zipcode", column=@Column(name="work_zipcode"))
})
private Address workAddress;
```

## ▷ 값 타입 컬렉션
값 타입 컬렉션은 @ElementCollection을 이용해 엔티티 안에서 복수의 값 객체
(Embeddable 타입 또는 기본 타입)를 컬렉션으로 관리할 수 있게 해주는 기능이다.

### ✅ 값 타입 컬렉션이란?
- 값 타입(@Embeddable 또는 기본 타입)을 컬렉션(List, Set, Map 등) 으로 다룰 수 있게 해줌.
- 별도의 테이블을 만들어서 1:N 구조로 매핑.
- 값 타입은 엔티티가 아니므로 식별자(Primary Key)가 없음.
- 부모 엔티티에 완전히 종속됨. (Life-cycle 의존적)
- 값 타입 컬렉션도 지연 로딩 전략 사용.

### ✨ 예시 코드:
```java
// 기본 타입 컬렉션
@ElementCollection
@CollectionTable(name = "FAVORITE_FOODS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME")
private Set<String> favoriteFoods = new HashSet<>();
```
```java
// 임베디드 값 타입 컬렉션
@Embeddable
public class Address {
  private String city;
  private String street;
  private String zipcode;
}

@ElementCollection
@CollectionTable(name = "ADDRESS_HISTORY", joinColumns = @JoinColumn(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<>();
```

### ❌ 값 타입 컬렉션의 제약사항
- 값은 변경하면 추적이 어렵다.
- 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 
값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.
(null 입력X, 중복 저장X)

### 💡 값 타입 컬렉션 대안
- 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려.
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용.
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬 렉션 처럼 사용.

### ✨ 예시 코드:
```java
@Entity
public class AddressEntity {
    @Id @GeneratedValue
    private Long id;
    private Address address; // 임베디드 타입
}

@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "MEMBER_ID")
private List<AddressEntity> addressHistory = new ArrayList<>();
```
실무에서는 위 코드처럼 일대다 관계를 고려하는 것이 필요하다.

### 값 타입 컬레션 언제 쓰면 좋을까?
- 단순한 값들의 모음일 때 (예: String, Enum, 작은 VO 등)
- 자주 변경되지 않는 값들일 때
- 해당 값에 ID, 관계, 로직이 필요 없는 경우


