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


## 다양한 의존관계 주입 방법

- 생성자 주입
- 수정자 주입 (setter)
- 필드 주입
- 일반 메서드 주입

### 생성자 주입

- 생성자 호출 시점에 딱 한 번만 호출되는 것을 보장
- 불변, 필수 의존 관계에 적용
- 생성자가 한 개만 있으면 @Autowired 애노테이션 생략 가능

```java
@Component
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

### 수정자 주입

- 선택, 변경 가능성이 있는 의존관계에 적용
- 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용

```java
@Component
public class OrderServiceImpl implements OrderService {

  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;

  @Autowired
  public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }

  @Autowired
  public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
  }
}
```

### 필드 주입

- 사용 권장 XXXXXX
  - 테스트 코드나 @Configuration 같은 곳에서만 사용
- 간결해서 좋지만 외부에서 변경이 불가능해 테스트하기가 힘들다

```java
@Component
public class OrderServiceImpl implements OrderService {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private DiscountPolicy discountPolicy;
    
}
```

### 일반 메서드 주입

- 한 번에 여러 필드를 주입 받을 수 있다
- 일반적으로 잘 사용하지 XXXX

```java
@Component
public class OrderServiceImpl implements OrderService {

  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;

  @Autowired
  public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

## 자동, 수동의 올바른 실무 운영 기준

### 편리한 자동 기능을 기본으로 사용하자

애플리케이션은 업무 로직과 기술 지원 로직으로 나눌 수 있다
- 업무 로직 빈 : 웹을 지원하는 컨트롤러, 서비스, 레파지토리 등이 업무 로직이다. 비즈니스 요구사항을 개발할 때 추가되거나 변경
- 기술 지원 로직 빈 : 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용. 데이터베이스 연결이나 공통 로그 처리같은 업무 로직을 지원하기 위한 하부기술, 공통 기술
- 업무 로직은 어느정도 유사한 패턴이 있어 자동 빈 등록으로 사용해도 무관
- 기술 지원 로직은 업무 로직에 비해 수가 적고, 앱 전반에 걸쳐 영향을 미친다. 업무 로직은 어디가 문제인지 명확하지만, 기술 지원 로직은 파악하기 어렵다. 이런 기술 지원 로직은 수동 빈 등록을 사용해서 명확하게 드러내는 게 좋다.

### 비즈니스 로직 중 다형성을 활용할 때

- 내가 한 추상화를 다른 개발자가 보면 이해할 수 없을 수도 있다. 이해하기 쉽게 특정 패키지에 같이 묶어둔다.
