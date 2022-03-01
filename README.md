# 영한님 스프링 핵심 원리 - 기본편

## @Configuration, 바이트 코드 조작 마법

- AppConfig 클래스에 @Configuration 애노테이션이 있다.
- AppConfig의 스프링 빈을 조회하여 클래스 정보를 출력해보면 `AppConfig$$EnhancerBySpring ...` 같이 순수한 클래스 정보가 출력되지 않는다.
- 이것은 내가 만든 클래스가 아니라 스프링이 `CGLIB`이라는 바이트코드 조작 라이브러리를 이용해 AppConfig 클래스를 상속 받은 임의의 다른 클래스를 만들어 스프링 빈에 등록한 것이다.
- 이 임의의 다른 클래스가 싱글톤을 보장하게 해준다.
- @Configuration을 주석처리하면 싱글톤이 보장되지 않는 테스트 결과를 확인할 수 있었다.
- 스프링 설정 정보는 항상 `@Configuration` 을 사용하자.