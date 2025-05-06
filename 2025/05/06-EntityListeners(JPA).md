# 📚 2025.05.06 TIL
## 실용적인 테스트 가이드 (박우빈)
##  ◆ EntityListeners 어노테이션
```@EntityListeners(AuditingEntityListener.class)```가 동작하는
원리는 JPA의 엔티티 생명주기 콜백 + 스프링의 이벤트 처리를 활용한 구조이다.

---

### ▷ 전체 동작 원리 요약
1. ```@EntityListeners```는 JPA의 콜백 리스너 설정 어노테이션이다.
2. ```AuditingEntityListener```는 JPA 콜백을 감지해서 Spring의 ```AuditorAware```와 함께 동작한다.
3. ```@CreatedDate```, ```@LastModifiedDate``` 같은 필드에 자동으로 값이 설정된다.

---

### ✅ 단계별 설명
### 1. @EntityListeners(...)
```java
@EntityListeners(AuditingEntityListener.class)
```
- 이건 JPA 기능으로, 엔티티의 생명주기 이벤트(```@PrePersist```, ```@PreUpdate``` 등)가 
발생할 때 지정한 클래스의 메서드를 호출하겠다는 선언이다.

### 2. ```AuditingEntityListener``` 클래스 내부
- 이 클래스는 ```@PrePersist```, ```@PreUpdate``` 시점에 호출된다.
- 그 안에서 해당 엔티티에 붙은 @CreatedDate, @LastModifiedDate 등의 필드를 
reflection 으로 찾아서 값을 자동으로 채운다.

```java
@PrePersist
public void touchForCreate(Object target) {
    // target의 @CreatedDate 필드 찾아서 LocalDateTime.now() 등으로 설정
}

@PreUpdate
public void touchForUpdate(Object target) {
    // target의 @LastModifiedDate 필드 업데이트
}
```

### 3. ```@EnableJpaAuditing```의 역할
- 이 설정은 Spring이 AuditingEntityListener가 사용할 수 있는
DateTimeProvider나 AuditorAware 빈을 주입해줄 수 있게 활성화한다. 
즉, Spring이 AuditingEntityListener에게 “지금은 생성 시점이다” 라고 알려주는 뇌 역할을 하게된다.

### 4. 필드에 붙은 어노테이션
```java
@CreatedDate
@Column(updatable = false)
private LocalDateTime createdDateTime;

@LastModifiedDate
private LocalDateTime modifiedDateTime;
```
#### 이 필드들이 엔티티 저장/수정 시점에 자동으로 채워지는 이유는:
- JPA 생명주기 콜백이 발생하면서
- ```AuditingEntityListener```가 필드를 relection 으로 분석해서
- 현재 시간을 넣어주기 때문이다.

--- 

### ▷ 전체 흐름 요약
```less
@PrePersist / @PreUpdate (JPA 콜백)
        ↓
AuditingEntityListener
        ↓
필드 스캔 (@CreatedDate, @LastModifiedDate)
        ↓
현재 시간 or 현재 사용자 채워넣기
```
