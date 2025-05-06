# 📚 2025.04.16 TIL
## 실전! 스프링 부트와 JPA 활용1 (김영한)
##  ◆ 변경 감지와 병합(merge)

## ▷ 변경 감지
변경 감지(Dirty Checking)는 영속성 컨텍스트가 관리하는 엔티티 객체의 필드 값이 
변경되었을 때, 자동으로 이를 감지하여 DB에 반영하는 기능이다.

### 📌 동작 원리
1. 엔티티가 영속 상태가 되면 (```persist()```로 저장되거나, 조회된 객체를 사용할 때) 
JPA는 해당 객체의 스냅샷(초기 상태 복사본)을 만들어 둔다.
2. 트랜잭션 안에서 엔티티의 값을 변경하면 변경된 엔티티와 스냅샷을 비교한다.
3. 트랜잭션을 커밋하거나 ```flush()```를 호출하면 변경 사항이 감지되고 자동으로
```UPDATE```쿼리가 발생한다.

### ✨ 예시 코드
```java
@Transactional
public void updateItem(Long itemId, int newPrice) {
    Item item = em.find(Item.class, itemId); // 1. 영속 상태로 진입

    item.setPrice(newPrice); // 2. 값 변경

    // 3. 트랜잭션 커밋 시 변경 감지 → UPDATE 쿼리 자동 실행
}
```
### 변경 감지가 일어나는 조건
- 해당 엔티티는 영속성 컨텍스트에 의해 관리되고 있어야 함.
- 변경 감지는 트랜잭션 안에서 일어나고, 커밋 시점에 반영됨.
- 단순 setter 호출 등으로 엔티티 필드가 바뀌어야 감지됨.

## ▷ 병합(merge)
```merge()```는 준영속 엔티티나 비영속 엔티티를 영속성 컨텍스트에 복사해서,
새로운 영속 엔티티를 반환하는 메서드이다.

### ✅ 준영속 엔티티란?
준영속 엔티티는 DB에 저장된 적은 있지만 현재는 영속성 컨텍스트에서 관리되지 않는 객체를 뜻한다.

### ✨ 준영속 엔티티 예시 코드:
```java
// 실제 DB에 이미 저장된 Book (id=1)
Book book = new Book();
book.setId(1L); // 식별자 수동 지정
book.setName("New Name"); // 값 변경
```
- 위에서 만든 ```book```객체는 new로 만들었기 때문에 원래는 비영속이지만,
식별자(id)가 있기 때문에 JPA는 이를 준영속 엔티티로 본다.
- 따라서 이후 setName은 JPA가 변경 감지를 하지 못하게 된다.

### ✨ 병합 예시 코드:
```java
@Transactional
public void updateBook() {
    Book book = new Book();
    book.setId(1L); // DB에 이미 존재하는 ID
    book.setName("Updated Title");

    Book mergedBook = em.merge(book); // 새로운 영속 객체로 병합

    System.out.println(em.contains(book));        // false (준영속)
    System.out.println(em.contains(mergedBook));  // true (영속)
}
```
- 위 코드에서 ```book```은 여전히 준영속 상태로 영속성 컨텍스트에서 관리되지 않는다.
- 반환된 ```mergedBook```은 영속 상태이고, 실제 DB와 동기화됨.

### 📌 내부 동작 순서 (merge 시 JPA 내부 흐름)
1. 식별자 기준으로 DB에서 기존 엔티티를 조회 (```find()``` 수행)
2. 조회된 영속 엔티티에 준영속 엔티티의 값들을 복사
3. 복사된 엔티티를 영속성 컨텍스트에서 관리
4. 트랜잭션 커밋 시, 변경된 값이 DB에 반영 (```UPDATE``` 쿼리)

### ❗주의:
변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면
모든 속성이 변경된다. 병합시 값이 없으면 ```null```로 업데이트 할 위험도 있다.
따라서 실무에서는 병합보다는 변경 감지를 사용하는 것을 추천한다.
