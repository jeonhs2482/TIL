# 2025.04.05 TIL
## 김영한의 실전 자바 - 중급 2편
##  ◆ 제네릭 - Generic
제네릭은 클래스나 메서드에서 사용할 데이터 타입을 파라미터화할 수 있게 해주는
기능이다. 즉, 실제 사용할 타입을 나중에 지정할 수 있도록 만들어 준다.

### 1. 제네릭을 사용하는 이유
- 컴파일 시 타입 체크 가능 -> 안정성 증가
    - 잘못된 타입 사용 시 컴파일 오류 발생.
    - 런타임 오류 줄어듬.
- 형변환 필요 없음 -> 코드 간결
    - Object를 쓸 때처럼 형변환(down casting)을 하지 않아도 됨.
- 재사용성 증가
    - 하나의 코드로 여러 타입을 처리 가능.

### ✨ 예시 코드:
```java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```
```java
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String str = stringBox.get(); // 형변환 필요 없음
// Object 사용 시 Object str = (String) stringBox.get() 형변환 해야함.

Box<Integer> intBox = new Box<>();
intBox.set(123);
Integer num = intBox.get();
```
#### ✅ 제네릭의 핵심은 사용할 타입을 미리 결정하지 않는다는 점이다. 클래스 내부에서 사용하는 타입을 클래스를 정의하는 시점에 결정하는 것이 아니라 실제 사용하는 생성 시점에 타입을 결절하는 것이다.
