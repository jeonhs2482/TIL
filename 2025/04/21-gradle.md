# 📚 2025.04.21 TIL
## Gradle for java developer (udemy)

## ◆ Gradle이란?
Gradle은 빌드 자동화 도구로서, Java, Kotlin, Groovy 같은 언어로 작성된 
프로젝트에서 컴파일, 테스트, 패키징, 배포 등을 자동화해주는 역할을 한다.

## 1. Gradle 특징
### 🎯 Gradle은 "Convention over Configuration"을 사용한다.
즉, "개발자가 모든 걸 일일이 설정하지 않아도, 기본 규칙(관례)을 따라 자동으로 동작한다"는 철학이다.

### ✅ 예1: src/main/java
```css
src/
 └── main/
     └── java/
```
→ 따로 설정하지 않아도 이 위치에 파일이 있으면 자동으로 컴파일함!

### ✅ 예2: build.gradle에 java 플러그인만 추가
```groovy
plugins {
    id 'java'
}
```
이렇게만 적으면,
- 소스 경로는 src/main/java
- 테스트 코드는 src/test/java
- 빌드 결과는 build/ 폴더에 생성 </br>
→ 전부 자동 설정됨 (관례 기반) 

## 2. Gradle Wrapper
Gradle Wrapper는 Gradle을 로컬에 설치하지 않아도 프로젝트를 빌드할 수 있게 해준다.

### 🛠️ 왜 필요할까?
Gradle은 버전이 자주 바뀌고, 프로젝트마다 호환되는 버전이 다를 수 있다. </br>
→ 팀원마다 Gradle 버전이 다르면 빌드 오류가 날 수도 있음
#### Gradle Wrapper를 사용하면:
- 필요한 Gradle 버전을 자동으로 다운로드
- 해당 버전으로 빌드 실행
- 전 팀원이 동일한 환경에서 작업 가능
- 
### ✅ 공유해야 할 파일들
```pgsql
gradlew                ← 유닉스/macOS 실행 스크립트
gradlew.bat            ← 윈도우용 실행 스크립트
gradle/wrapper/
 ├── gradle-wrapper.jar         ← 실행에 필요한 자바 코드
 └── gradle-wrapper.properties  ← 사용할 Gradle 버전 명시
```
위 파일들만 있으먄, 팀원은 아래처럼 실행할 수 있음:
```bash
# Gradle 설치 없이 바로 실행
./gradlew build
```
→ 필요한 Gradle 버전이 자동으로 다운로드되고, 빌드도 그 버전으로 실행됨.

## 3. build.gradle 파일 구성

### 📌 플러그인 블록
build.gradle 파일에서 Gradle이 프로젝트에 어떤 기능을 추가할지 정의하는 
영역이 바로 plugins { ... } 블록이다.
### 예를 들어:
```grrovy
plugins {
    id 'java'
}
```
이렇게 작성하면 Gradle이 해당 프로젝트를 Java 프로젝트로 인식하고 자동으로 컴파일,
테스트, JAR 생성 같은 Java 관련 작업을 추가해준다.

