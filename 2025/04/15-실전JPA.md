# 📚 2025.04.14 TIL
## 실전! 스프링 부트와 JPA 활용1 (김영한)
##  ◆ 실전 JPA 활용
## 1. RequiredArgsConstructor
```@RequiredArgsConstructor```는 Lombok에서 제공하는 어노테이션으로
주로 생성자 자동 생성을 위해 사용된다.

### 📌 일반적으로 우리가 작성해야 하는 코드:
```java
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
- 테스트 코드 작성 시 같은 경우에 ```memberRepository```에 다른 객체 등을
주입해야 하는 경우가 있을 수 있으므로 생성자를 통해 주입하는 것이 필요하다.
- 하지만 위와 같이 코드를 작성하게 되면 작성해야하는 코드가 많아지게된다.

### 📌 @RequiredArgsConstructor를 쓰면:
```java
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    // 생성자 자동 생성됨!
}
```
- final 필드를 기반으로 생성자를 Lombok이 자동 생성해준다.
- final이나 @NonNull이 붙은 필드만 파라미터로 받는 생성자 생성.


