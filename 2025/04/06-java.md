# 📚 2025.04.06 TIL
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

### 2. 타입 매개변수 제한
제네릭은 클래스나 메서드를 정의할 때 사용할 타입을 외부에서 지정할 수 있게 해주는 기능이다.
그런데 원하는 기능에 모든 타입이 다 의미 있는 건 아니기 때문에, 타입 매개변수에 제한을 둘 수 있다.

### ✨ 예시 코드:
```java
class Animal {
    public void sound() {
        System.out.println("Some animal sound");
    }
}

class Dog extends Animal {
    public void sound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    public void sound() {
        System.out.println("Meow");
    }
}
```
```java
class AnimalCage<T extends Animal> {
    private T animal;

    public void setAnimal(T animal) {
        this.animal = animal;
    }

    public void makeSound() {
        animal.sound();
    }
}
```
- 타입 매개변수로 ```T extends Animal```을 선언한 덕분에 T는 반드시 Animal 또는 그 하위 클래스만 허용.
- 따라서 타입 매개변수로 ```T```만 선언했을때는 사용할 수 없었던 ```animal.sound()``` 메서드를 호출할 수 있게 됨.

### 3. 제네릭 메서드
제네릭 메서드는 메서드 자체에 타입 매개변수를 선언하는 것이다.
즉, 클래스 전체가 제네릭이 아니더라도, 특정 메서드만 제네릭으로 만들 수 있다.

### ✨ 예시 코드:
```java
public class GenericMethod {
    public static <T> T genericMethod(T t) {
        System.out.println("generic print: " + t);
        return t;
    }

    public static <T extends Number> T numberMethod(T t) {
        System.out.println("bound print: " + t);
        return t;
    }
}
```
```java
public class MethodMain1 {
    public static void main(String[] args) {
        Integer i = 10;

        // 타입 인자(Type Argument) 명시적 전달
        System.out.println("명시적 타입 인자 전달");
        Integer result = GenericMethod.<Integer>genericMethod(i);
        Integer integerValue = GenericMethod.<Integer>numberMethod(10);
        Double doubleValue = GenericMethod.<Double>numberMethod(20.0);

        //타입 추론, 타입 인자 생략
        System.out.println("타입 추론");
        Integer result2 = GenericMethod.genericMethod(i);
        Integer integerValue2 = GenericMethod.numberMethod(10);
        Double doubleValue2 = GenericMethod.numberMethod(20.0);
    }
}
```
- 제네릭 타입은 객체를 생성하는 시점에 타입 인자를 전달하고, 제네릭 메서드는 메서드를 호출하는 시점에 타입 인자를 전달한다.
- 제네릭 메서드는 클래스 전체가 아니라 특정 메서드 단위로 제네릭을 도입할 때 사용한다.
- 제네릭 메서드의 핵심은 메서드를 호출하는 시점에 타입 인자를 전달해서 타입을 지정하는 것이다. 
따라서 타입을 지정하면서 메서드를 호출한다.

### 💡 참고
제네릭 타입은 static 메서드에 타입 매개변수를 사용할 수 없다. 제네릭 타입은 객체를 생성하는 시점에
타입이 정해진다. 그런데 static 메서드는 인스턴스 단위가 아니라 클래스 단위로 작동하기 때문에
제네릭 타입과는 무관하다. 따라서 static 메서드에 제네릭을 도입하려면 제네릭 메서드를 사용해야 한다.

### ✨ 예시 코드:
```java
class Box<T> {
T instanceMethod(T t) {} //가능
static T staticMethod1(T t) {} //제네릭 타입의 T 사용 불가능
static <V> V staticMethod2(V t) {} //static 메서드에 제네릭 메서드 도입  
}
```

### 4. 와일드카드
와일드카드란 제네릭 타입에서 정확한 타입 대신 ?(물음표)를 사용해서 어떤 타입이든
받아들일 수 있게 만든 것이다.

### 💡 참고
와일드카드는 제네릭 타입이나, 제네릭 메서드를 선언하는 것이 아니라 이미 만들어진 제네릭 타입을 활용할 때 사용한다.

### 📌 비 제한 와일드카드:
```java
//이것은 제네릭 메서드이다.
//Box<Dog> dogBox를 전달한다. 타입 추론에 의해 타입 T가 Dog가 된다.
static <T> void printGenericV1(Box<T> box) {
  System.out.println("T = " + box.get());
}

//이것은 제네릭 메서드가 아니다. 일반적인 메서드이다.
//Box<Dog> dogBox를 전달한다. 와일드카드 ?는 모든 타입을 받을 수 있다.
static void printWildcardV1(Box<?> box) {
  System.out.println("? = " + box.get());
}
```

### 📌 상한 와일드카드:
```java
static <T extends Animal> void printGenericV2(Box<T> box) {
T t = box.get();
System.out.println("이름 = " + t.getName());
}

static void printWildcardV2(Box<? extends Animal> box) {
Animal animal = box.get();
System.out.println("이름 = " + animal.getName());
}
```

### 📌 하한 와일드카드:
```java
static void writeBox(Box<? super Animal> box) {
  box.set(new Dog("멍멍이", 100));
}

// Animal 포함 상위 타입 전달 가능
writeBox(objBox);
writeBox(animalBox);
//writeBox(dogBox); // 하한이 Animal 전달 불가
//writeBox(catBox); // 하한이 Animal 전달 불가
```

### ⚠️ 타입 매개변수가 꼭 필요한 경우
와일드카드는 제네릭을 정의할 때 사용하는 것이 아니다. Box<Dog>, Box<Cat>처럼 타입 인자가
전달된 제네릭 타입을 활용할 때 사용한다. 따라서 다음과 같은 경우에는 제네릭 타입이나 제네릭 메서드를
사용해야 문제를 해결할 수 있다.

### ✨ 예시 코드:
```java
static <T extends Animal> T printAndReturnGeneric(Box<T> box) {
  T t = box.get();
  System.out.println("이름 = " + t.getName());
  return t;
}

static Animal printAndReturnWildcard(Box<? extends Animal> box) {
  Animal animal = box.get();
  System.out.println("이름 = " + animal.getName());
  return animal;
}

// printAndReturnGeneric() 은 다음과 같이 전달한 타입을 명확하게 반환할 수 있다.
Dog dog = WildcardEx.printAndReturnGeneric(dogBox);

// printAndReturnWildcard() 의 경우 전달한 타입을 명확하게 반환할 수 없다.
// 여기서는 `Animal` 타입으로 반환한다.
Animal animal = WildcardEx.printAndReturnWildcard(dogBox);
```

메서드의 타입들을 특정 시점에 변경하려면 제네릭 타입이나, 제네릭 메서드를 사용해야 한다.
와일드카드는 이미 만들어진 제네릭 타입을 전달 받아서 활용할 때 사용한다. 따라서 메서드의 타입들을
타입 인자를 통해 변경할 수 없다. 쉽게 이야기해서 일반적인 메서드에 사용한다고 생각하면 된다.