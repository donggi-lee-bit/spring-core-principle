# 영한님 스프링 핵심 원리 - 기본편

## @Configuration, 바이트 코드 조작 마법

- AppConfig 클래스에 @Configuration 애노테이션이 있다.
- AppConfig의 스프링 빈을 조회하여 클래스 정보를 출력해보면 `AppConfig$$EnhancerBySpring ...` 같이 순수한 클래스 정보가 출력되지 않는다.
- 이것은 내가 만든 클래스가 아니라 스프링이 `CGLIB`이라는 바이트코드 조작 라이브러리를 이용해 AppConfig 클래스를 상속 받은 임의의 다른 클래스를 만들어 스프링 빈에 등록한 것이다.
- 이 임의의 다른 클래스가 싱글톤을 보장하게 해준다.
- @Configuration을 주석처리하면 싱글톤이 보장되지 않는 테스트 결과를 확인할 수 있었다.
- 스프링 설정 정보는 항상 `@Configuration` 을 사용하자.

## @ComponentScan

### 탐색할 패키지의 시작 위치 지정

- 모든 컴포넌트 스캔을 하면 시간이 오래 걸려 필요한 위치부터 탐색시킬 수 있다.

```java
@ComponentScan (
        basePackages = "hello.core"
)
```

- 하지만 권장하는 방법은 패키지 위치를 지정하지 않고 설정 정보 클래스를 프로젝트 최상단에 두게한다.

### 애노테이션 부가 기능

- @Controller
  - 스프링 mvc 컨트롤러로 인식
- @Repository
  - 스프링 데이터 접근 계층으로 인식
  - 데이터 게층의 예외를 스프링 예외로 변환
- @Configuration
  - 스프링 설정 정보 인식
  - 스프링 빈이 싱글톤을 유지하도록 추가 처리
- @Service
  - 특별한 처리 하지 X
  - 개발자들이 핵심 비즈니스 로직이 여기 있겠다싶어 비즈니스 계층을 인식시켜줌

## 스프링 빈 중복 등록과 충돌

1. 자동 빈 등록 vs 자동 빈 등록
2. 수동 빈 등록 vs 자동 빈 등록

### 자동 빈 등록 vs 자동 빈 등록

- 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데 이 때 이름이 같은 경우 오류 발생
- `ConflictingBeanDefinitionException` 예외 발생

### 수동 빈 등록 vs 자동 빈 등록

- 수동 등록 빈이 우선권을 가지게 된다 -> 수동 빈이 자동 빈을 `오버라이딩`
- 의도해서 하는 경우라기 보다 설정이 꼬여서 일어난 일일 확률이 높다.
  - 이런 게 잡기 힘든 어려운 버그가 됨
- 스프링 부트에서 수동 빈 등록과 자동 빈 등록 충돌이 나면 오류가 발생하도록 기본 값을 변경