## 📚 스프링 핵심 원리 - 고급 1편 (김영한)
##  ◆ 스레드 join 과 인터럽트
## 📌 스레드의 join 메서드
### 정의:
```Thread.join()``` 메서드는 멀티스레딩 환경에서 특정 스레드의 실행이 완료될 때까지 현재 스레드의 실행을 일시 중지시키는 역할을 한다.
즉, "다른 스레드가 끝날 때까지 기다린다"는 의미이다.

### 주요 특징:
1) 스레드 동기화 
- ```join()``` 메서드를 사용하면 스레드 간의 실행 순서를 제어할 수 있다. 예를 들어, 메인 스레드에서 여러 작업 스레드를 시작한 후, 
모든 작업 스레드가 완료될 때까지 메인 스레드가 다음 작업을 진행하지 않도록 할 때 유용하다.
- 작업 스레드들이 어떤 데이터를 생성하고, 메인 스레드가 그 데이터를 모두 취합해서 최종 결과를 만들어야 하는 경우에 ```join()```을 사용하여 
모든 작업 스레드가 데이터를 생성할 때까지 기다릴 수 있다.

2) 실행 흐름 제어
- ```join()```을 호출한 스레드는 호출 대상 스레드가 종료(정상 종료 또는 예외 발생)되거나, 지정된 시간이 경과할 때까지 블록(대기)된다.

3) 오버로드된 메서드
- 대부분의 언어에서 ```join()``` 메서드는 여러 오버로드된 형태로 제공된다.
- join() (인자 없음): 대상 스레드가 종료될 때까지 무기한으로 기다린다.
- join(long millis) 또는 join(float timeout): 지정된 시간(밀리초 또는 초) 동안만 기다린다. 
시간이 만료되거나 대상 스레드가 종료되면 호출 스레드는 대기를 멈추고 다음 작업을 진행한다.

4) 예외 처리
- 대기 중인 스레드가 ```InterruptedException``` 과 같은 예외에 의해 중단될 수 있다. 
따라서 ```join()``` 호출은 일반적으로 try-catch 블록으로 감싸서 예외를 처리해야 한다.

### ✨ 예시:
```java
public class ThreadJoinExample {
    public static void main(String[] args) {
        Thread workerThread = new Thread(() -> {
            try {
                System.out.println("작업 스레드 시작...");
                Thread.sleep(2000); // 2초 동안 작업 수행
                System.out.println("작업 스레드 완료.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        workerThread.start(); // 작업 스레드 시작

        System.out.println("메인 스레드: 작업 스레드가 완료될 때까지 기다립니다.");
        try {
            workerThread.join(); // 작업 스레드가 완료될 때까지 메인 스레드 대기
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("메인 스레드: 작업 스레드 완료를 확인했습니다. 다음 작업을 진행합니다.");
    }
}
```
- 위 예시에서 main 스레드는 ```workerThread```를 시작한 후 ```workerThread.join()```을 호출하여 ```workerThread```가 2초간의 작업을 마칠 때까지 기다린다.
```join()```이 없었다면 main 스레드는 ```workerThread```의 완료 여부와 상관없이 즉시 다음 "메인 스레드: 작업 스레드 완료를 확인했습니다." 메시지를 출력했을 것이다.

---

## 📌 스레드의 인터럽트
### 정의:
스레드 인터럽트는 스레드에게 "현재 하던 작업을 멈추고 다른 일을 하거나, 종료하라"고 요청하는 신호를 보내는 것이다. 
중요한 점은 이 신호가 스레드를 강제로 종료시키거나 즉시 멈추게 하는 것이 아니라, 스레드 스스로가 이 신호를 인지하고 
적절히 반응하도록 유도하는 방식이라는 것이다.

### 스레드 인터럽트의 핵심 개념:
1) 인터럽트 플래그
- 모든 자바 스레드는 내부에 ```interrupted```라는 ```boolean``` 타입의 플래그를 가지고 있다. 기본값은 ```false```이다.
- ```Thread.interrupt()``` 메서드를 호출하면, 대상 스레드의 이 ```interrupted``` 플래그가 ```true```로 설정된다.

2) InterruptedException
- 자바에는 스레드를 대기 상태로 만드는 몇몇 메서드들(예: Thread.sleep(), Object.wait(), Thread.join(), BlockingQueue.put()/take(), Lock.lockInterruptibly() 등)이 있다.
- 이 메서드들은 스레드가 블록되어 있는 동안 ```interrupt()``` 호출을 받으면, 블록 상태에서 벗어나면서 ```InterruptedException```을 발생시킨다.
- ```InterruptedException```이 발생하면 해당 스레드의 ```interrupted``` 플래그는 자동으로 ```false```로 재설정된다.

3) 인터럽트 상태 확인 메서드
- ```boolean isInterrupted()```: 대상 스레드의 현재 ```interrupted``` 플래그 상태를 반환한다. 플래그 상태를 변경하지 않는다.
- ```static boolean Thread.interrupted()```: 현재 실행 중인 스레드의 ```interrupted``` 플래그 상태를 반환하고, 동시에 ```interrupted``` 플래그를 ```false```로 재설정한다.
(static 메서드이며 플래그를 초기화하는 역할이 있음)

### ✨ 예시:
```java
public class InterruptionExample1 {
    public static void main(String[] args) throws InterruptedException {
        Thread sleepyThread = new Thread(() -> {
            try {
                System.out.println("Sleepy thread: 5초간 잠듭니다...");
                Thread.sleep(5000); // 5초간 대기
                System.out.println("Sleepy thread: 잠에서 깨어났습니다.");
            } catch (InterruptedException e) {
                System.out.println("Sleepy thread: 인터럽트 받아서 잠에서 깨어났습니다!");
                // 인터럽트 발생 시 처리할 로직 (자원 정리 등)
                // 중요: InterruptedException 발생 시 플래그는 자동으로 false가 됨.
                // 따라서 만약 이 인터럽트 상태를 상위 호출자에게 전달하고 싶다면,
                // 다시 Thread.currentThread().interrupt(); 를 호출하여 플래그를 설정해야 함.
                Thread.currentThread().interrupt(); // 다시 플래그를 설정하여 상위 호출자에게 알림
            }
            System.out.println("Sleepy thread: 작업 종료.");
        });

        sleepyThread.start();
        Thread.sleep(1000); // 메인 스레드 1초 대기
        System.out.println("Main thread: Sleepy thread를 인터럽트합니다.");
        sleepyThread.interrupt(); // Sleepy thread에게 인터럽트 신호 보냄
    }
}
```

--- 

