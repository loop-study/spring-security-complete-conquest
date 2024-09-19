# 초기화 과정 이해
### 프로젝트 생성 / 의존성 추가

### SecurityBuilder / SecurityConfigurer

### WebSecurity / HttpSecurity
#### HttpSecurity
- HttpSecurityConfigurer에서 HttpSecurity를 생성하고 초기화한다.
- HttpSecurity는 보안에 필요한 각 설정 클래스와 필터들을 생성하고 최종적으로 SecurityFilterChain 빈 생성
    - SecurityFilterChain 생성하는 것이 최종 목적이다.
      - requestMatcher, filters 를 가진다.
    - 각 필터가 연결되어있다고 해서 필터체인이라 부른다.

#### SecurityFilterChain
- matches 메서드를 통해 여러 필터체인을 가지는 걸 알 수 있다.
- getFilters 메서드를 통해 필터 체인을 확인한다. 각 필터는 요청 처리 과정에서 특정 작업 (인증, 권한 부여 로깅 등)을 수행한다.

#### WebSecurity
- WebSecurityConfiguration에서 WebSecurity 생성하고 초기화를 진행
- WebSecurity는 HttpSecurity 에서 생성한 SecurityFilterChain 빈을 SecurityBuilder에 저장
- WebSecurity가 build()를 실행하면 SecurityBuilder에서 SecurityFilterChain를 꺼내 FilterChainProxy 생성자에 전달

### DelegatingFilterProxy / FilterChainProxy
#### Filter
- 서블릿 필터는 웹 애플리케이션에서 클라이언트의 요청과 서버의 응답을 가공(요청과 응답을 변경가능)하거나 검사하는데 사용되는 구성 요소
- 서블릿 필터는 클라이언트의 요청이 서블릿에 도달하기 전이나 서블릿이 응답을 클라이언트에게 보내기 전에 특정 작업을 수행할 수 있다.
- 서블릿 필터는 서블릿 컨테이너(WAS)에서 생성되고 실행되고 종료된다.

#### DelegatingFilterProxy
- DelegatingFilterProxy는 스프링에서 사용되는 '특별한' 서블릿 필터로 서블릿 컨테이너와 스프링 애플리케이션 컨텍스트 간의 연결고리 역할을 하는 필터이다.
- DelegatingFilterProxy는 서블릿 필터의 기능을 수행하는 동시에 스프링의 의존성 주입 및 빈 관리 기능과 연동되도록 설계된 필터다
- DelegatingFilterProxy는 'springSecurityFilterChain' 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임한다.
- 실제 보안 처리를 수행하지 않는다*
  - 그냥 DelegatingFilterProxy는 요청을 받아서 스프링의 springSecurityFilterChain 빈에게 위임하는거다. 

#### FilterChainProxy
- springSecurityFilterChain 의 이름으로 생성되는 필터 빈으로서 DelegatingFilterProxy으로부터 요청을 위임받고 보안 처리 역할을 한다.
- 내부적으로 하나 이상의 SecurityFilterChain 객체들을 가지고 있으며 요청 URL 정보를 기준으로 적절한 SecurityFilterChain을 선택하여 필터를 호출
- HttpSecurity를 통해 API 추가 시 관련 필터들이 추가된다.
- 사용자의 요청을 필터 순서대로 호출함으로 보안 기능을 동작시키고 필요 시 직접 필터를 생성해서 기존의 필터 전,후로 추가 가능하다. 

### 사용자 정의 보안 설정하기
#### 사용자 정의 보안 기능 구현
- 한 개 이상의 SecurityFilterChain 타입의 빈을 정의한 후 인증 API 및 인가 API를 설정한다.
- @EnableWebSecurity를 반드시 선언해야한다.
  - 모든 설정 코드는 람다 형식으로 작성해야한다 (7.x 부터는 람다 형식만 지원 예정)
  - 수동으로 securityFilterChain 빈을 정의하면, 자동설정에 의한 securityFilterChain 빈은 생성되지 않는다.   

##### 사용자 추가 설정
- application.yml(혹은 프로퍼티) 파일로 추가할 수 있다.
  - spring:
    - security:
      - user:
        - name: user
        - password: 1111
        - roles: USER
- 자바 설정 클래스에 직접 정의도 가능하다.
