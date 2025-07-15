## 📚 스프링 핵심 원리 - 기본편 (김영한)
##  ◆ @PostConstruct 와 @PreDestroy
## 📌 빈(Bean)의 생명주기 관리 어노테이션
### 정의:
```@PostConstruct```와 ```@PreDestroy```는 자바 플랫폼의 표준 어노테이션으로, 스프링 프레임워크를 포함한 다양한 컨테이너에서 
빈(Bean)의 생명주기 이벤트를 관리하는 데 사용된다. 이 어노테이션들을 사용하면 빈의 초기화와 소멸 시점에 특정 로직을 수행하도록 지정할 수 있다.

### 📌@PostConstruct:
```@PostConstruct``` 어노테이션은 의존성 주입이 완료된 후, 빈이 초기화될 때 한 번만 호출되는 메서드에 붙인다.
생성자가 호출된 시점에는 아직 의존성 주입이 완료되지 않았을 수 있으므로, 주입된 의존성을 사용하여 초기화 로직을 수행해야 할 때 유용하다.

### 주요 사용사례:
- 주입받은 리소스를 사용하여 초기값 설정
- 캐시 데이터 미리 로딩
- 데이터베이스 연결 풀(Connection Pool) 초기화
- 애플리케이션 시작 시 필요한 설정값 로딩
### 실행 순서:
- 빈 생성자 호출
- 의존성 주입
- ```@PostConstruct``` 어노테이션이 붙은 초기화 메서드 호출

#### ✨예시:
```java
@Component
public class MyService {

    private MyDependency myDependency;

    // 의존성 주입
    public MyService(MyDependency myDependency) {
        this.myDependency = myDependency;
        System.out.println("생성자 호출");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct 호출: 의존성 주입 완료");
        // 주입받은 의존성을 사용한 초기화 작업 수행
        myDependency.prepare();
    }
}
```

---

### 📌@PreDestroy
```@PreDestroy``` 어노테이션은 스프링 컨테이너에서 빈이 제거되기 직전에 한 번만 호출되는 메서드에 붙인다. 
주로 빈이 사용하던 리소스를 해제하거나 종료 전에 처리해야 할 작업을 수행하는 데 사용된다.

### 주요 사용 사례:
- 데이터베이스 연결, 네트워크 소켓 등 리소스 해제
- 백그라운드 스레드 종료
- 임시 파일 삭제

### 실행 순서:
- 스프링 컨테이너 종료 시작
- ```@PreDestroy``` 어노테이션이 붙은 소멸 메서드 호출
- 빈 소멸

#### ✨예시:
```java
@Component
public class ResourceManager {

    private Connection connection;

    public ResourceManager() {
        // 리소스 획득 (예: DB 커넥션)
        this.connection = connectToDatabase();
        System.out.println("리소스 획득 완료.");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("@PreDestroy 호출: 리소스 해제 작업 수행");
        if (connection != null) {
            connection.close(); // 안전하게 리소스 해제
        }
    }
}
```

--- 




