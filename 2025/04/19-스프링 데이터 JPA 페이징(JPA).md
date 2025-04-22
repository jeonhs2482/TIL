# 📚 2025.04.19 TIL
## 실전! 스프링 데이터 JPA (김영한)
##  ◆ 스프링 데이터 JPA 페이징과 정렬
## 1. 순수 JPA 페이징과 정렬 
JPA에서 페이징을 어떻게 할 것인가? 다음 조건으로 페이징과 정렬을 사용하는 
예제 코드를 보자.

- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

### 📌 JPA 페이징 리포지토리 코드:
```java
public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by m.username desc", Member.class)
            .setParameter("age", age)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}

public long totalCount(int age) {
    return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
            .setParameter("age", age)
            .getSingleResult();
}
```
### 📌 JPA 페이징 테스트 코드:
```java
@Test
public void paging() throws Exception {
//given
memberJpaRepository.save(new Member("member1", 10));
memberJpaRepository.save(new Member("member2", 10));
memberJpaRepository.save(new Member("member3", 10));
memberJpaRepository.save(new Member("member4", 10));
memberJpaRepository.save(new Member("member5", 10));

int age = 10;
int offset = 0;
int limit = 3;

//when
List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
long totalCount = memberJpaRepository.totalCount(age);

//페이지 계산 공식 적용...
// totalPage = totalCount / size ...
// 마지막 페이지 ...
// 최초 페이지 ..
    
//then
assertThat(members.size()).isEqualTo(3);
assertThat(totalCount).isEqualTo(5);
}
```
순수 JPA에서는 위와 같이 데이터를 조회한 뒤 조회한 결과를 이용하여 페이징을 구현해야 한다. 
그럼 스프링 데이터 JPA에서는 페이징과 정렬을 어떤식으로 구현할 수 있는지 살펴보자.

## 2. 스프링 데이터 JPA 페이징과 정렬
### 페이징과 정렬 파라미터
- ```org.springframework.data.domain.Sort```: 정렬 기능 (정렬만 필요하고 페이징은 필요 없는 경우 사용)
- ```org.springframework.data.domain.Pageable```: 페이징 기능 (내부에 Sort 포함)

### 특별한 반환 타입
- ```org.springframework.data.domain.Page```: 추가 count 쿼리 결과를 포함하는 페이징
- ```org.springframework.data.domain.Slice```: 추가 count 쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit + 1 조회)
- ```List```(자바 컬렉션): 추가 count 쿼리 없이 결과만 반환

### 📌 Page 사용 예제 정의 코드:
```java
public interface MemberRepository extends Repository<Member, Long> {
    Page<Member> findByAge(int age, Pageable pageable);
}
```

### 📌 Page 사용 예제 실행 코드:
```java
//페이징 조건과 정렬 조건 설정
@Test
public void page() throws Exception {
//given
memberRepository.save(new Member("member1", 10));
memberRepository.save(new Member("member2", 10));
memberRepository.save(new Member("member3", 10));
memberRepository.save(new Member("member4", 10));
memberRepository.save(new Member("member5", 10));

//when
"username"));
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC,
Page<Member> page = memberRepository.findByAge(10, pageRequest);

//then
List<Member> content = page.getContent(); //조회된 데이터
assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
}
```
- ```findByAge()```에서 두번째 파라미터로 받은 ```Pageable```은 인터페이스다.
따라서 실제 사용할 때는 해당 인터페이스를 구현한 ```PageRequest```객체를 사용한다.
- ```PageRequest```생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를
입력한다. 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 페이지는 0부터 시작한다.

## 3. 카운트 쿼리 분리
카운트 쿼리는 DB에 있는 모든 데이터를 카운팅해야하기 때문에 데이터가 많아질수록 성능 이슈가 생길 수 있다.
이런점을 해결하기 위해 카운트 쿼리를 분리하여 작성할 수 있도록 하는 방법이 있다.

### ✅ 사용 예시:
```java
@Query(
  value = "select m from Member m left join m.team t",
  countQuery = "select count(m) from Member m"
)
Page<Member> findByAge(int age, Pageable pageable);

```
위 코드에서 처럼 기본적으로 ```Page<T>```를 반환할 경우, JPA는 내부적으로 count 쿼리도 
자동으로 생성한다. 하지만 ```value```쿼리에 ```join```, ```fetch join```, ```group by``` 등 
복잡한 로직이 있으면 자동 생성되는 count 쿼리도 복잡해져서 성능이 저하될 수 있다.
#### ➡ 이런 경우 카운트를 구하는 쿼리에서는 team을 join할 필요가 없기 때문에 (자동으로 카운트 쿼리 발생 시 join 쿼리 발생) 분리하여 작성하면 성능이 향상된다.








