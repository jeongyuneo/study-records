# AOP

## AOP

AOP(Aspect Oriented Programming)은 관점 지향 프로그래밍을 의미한다.

AOP는 핵심적인 비즈니스 로직으로부터 부가 기능을 분리하는 것에 목적을 둔다.

- 부가 기능 = 애플리케이션에서 코드가 중복되고, 강력하게 결합되어 있어 **다른 로직과 분리할 수 없는 애플리케이션 로직**

  ex) 트랜잭션, 로깅, 보안 등

### AOP 장점

1. 전체 코드 기반에 흩어져 있는 관심 사항이 하나의 장소로 응집 → **중복 제거**
2. 자신의 주요 관심 사항에 대한 코드만 포함하기 때문에 코드가 깔끔 → **가독성 향상**

⇒ 객체지향적으로 코드를 짤 수 있게 도우며 **유지보수가 용이**해짐

### AOP 용어

1. Target: 부가 기능을 부여할 대상
    - 클래스 혹은 객체
2. Advice: 타깃에게 제공할 부가 기능을 담은 모듈 = 부가 기능
    - **무슨 기능**을 **어느 시점**에 주입할 것인지 포함
3. Join Point: 어드바이스가 적용될 수 있는 위치
    - Spring AOP는 **메소드 실행 단계** 의미
4. Pointcut: 어드바이스를 적용할 조인 포인트를 선별하는 작업
    - Spring AOP는 **메소드**가 포인트컷의 대상
5. **Advisor**: 포인트컷과 어드바이스를 하나씩 갖고 있는 오브젝트
    - **어떤 메소드에 어느 시점에 어떤 기능**을 넣을 것인지 포함
6. **Weaving**: 조인 포인트에 어드바이스를 적용하는 방법
    - AOP가 핵심 기능(타겟)에 영향을 주지 않으면서 필요한 부가 기능(어드바이스)를 추가할 수 있도록 해주는 핵심적인 처리 과정
    - Spring AOP는 **Runtime Weaving** 사용

## Spring AOP

### Spring AOP 동작 과정

![](https://velog.velcdn.com/images/minide/post/bdc2b881-8aa7-4d7c-9b41-c0847a517b16/image.png)

1. 빈 객체 생성한 뒤 빈 후처리기에 전달
    - 빈 후처리기(BeanPostProcessor): 생성된 빈 객체를 스프링 컨테이너에 등록하기 전에 조작하는 객체
2. 어드바이저 내의 **포인트컷을 이용**해 전달받은 빈이 프록시 적용 대상인지 확인
3. 프록시 생성 대상 빈들은 프록시 객체 생성
4. 프록시 생성된 빈이면 프록시 객체를, 프록시 생성하지 않은 빈이면 그냥 빈 반환
5. 빈 후처리기에게 전달받은 객체를 컨테이너에 빈으로 등록

### Proxy

프록시(Proxy)는 원본 객체를 감싼 새로운 객체로, Spring AOP에서 부가 기능을 추가할 때 사용하는 패턴이다.

1. **JDK Dynamic Proxy**
    - 자바에서 제공하는 API(`java.lang.reflect.Proxy`)를 사용해 **인터페이스 기반**으로 Proxy 생성
        - 리플렉션의 `Proxy.newProxyInstance()` 메소드를 이용해 **동적으로 Proxy 생성** 가능
    - `InvocationHandler` 객체의 `invoke()` 메소드를 오버라이딩해 부가 기능 추가
    - 외부 라이브러리 의존하지 않음
2. CGLIB **(Code Generator Library)**
    - **클래스 기반**으로 **바이트 코드를 조작**해서 Proxy 생성
        - **메소드 재호출 시** 조작된 **바이트 코드 재사용**
        - 인터페이스가 아닌 타겟 객체를 직접 상속받아 만들기 때문에 `MethodMatcher` 객체를 사용해 특정 메소드만 프록시화하고, 나머지는 프록시를 거치지 않고 실제 객체 메소드를 호출하도록 만들
          수 있음
    - `MethodInterceptor`를 재정의한 `intercept()` 메소드를 구현해 부가 기능 추가
    - 타겟의 생성자를 두 번 호출
    - 외부 라이브러리(`net.sf.cglib.proxy.Enhancer`) 의존
    - 부모 클래스의 **기본 생성자** 필요(자식 클래스는 동적으로 생성)
    - 클래스 또는 메소드에 **final** 키워드 없어야 함(상속/오바리이딩 불가능하기 때문)

⇒ **Spring AOP는 JDK Dynamic Proxy가 default**지만, 인터페이스가 없는 경우 CGLIB 사용

⇒ **Spring Boot 2.0부터 CGLIB이 default**

Spring의 CGLIB 단점 보완

- 외부 라이브러리(`net.sf.cglib.proxy.Enhancer`) 의존 → since `Spring 3.2` Spring Core 패키지에 포함
- 부모 클래스의 기본 생성자 필요 & 타겟 생성자 두 번 호출 → since `Spring 4.0` CGLIB 프록시 인스턴스 `Objensis` 라이브러리를 통해 생성

### Spring AOP와 Proxy

- **Advice** = JDK Dynamic Proxy의 `invoke()` 메소드 내용 = CGLIB의 `intercept()` 메소드 내용
- **Join Point** = JDK Dynamic Proxy의 `InvocationHandler` = CGLIB의 `MethodInterceptor`
- **Pointcut** = JDK Dynamic Proxy의 조건문 = CGLIB의 `MethodMatcher`

### Weaving

Spring AOP는 RTW(Runtime Weaving)를 사용해 위빙한다.

- RTW(Runtime Weaving)
    - 런타임에 메소드 호출 시 프록시 객체를 생성해 실제 타겟 오브젝트의 변형없이 공통 기능 삽입
    - 포인트컷에 대한 어드바이스가 늘어날수록 성능이 떨어짐

## AspectJ

AspectJ는 Spring AOP 이외의 또다른 강력한 AOP 프레임워크 중 하나다.

AspectJ는 프록시를 이용하지 않고, GCLib를 이용해 타겟 오브젝트의 바이트 코드를 조작해 부가 기능을 직접 넣어주는 방식을 사용한다.

- AspectJ가 프록시를 사용하지 않고 바이트 코드를 조작하는 방법을 사용하는 이유
    1. 바이트 코드를 조작하면 Spring과 같은 컨테이너의 도움 필요 없음
    2. 프록시 방식보다 훨씬 강력하고 유연한 AOP 제공 가능

### Weaving

AspectJ는 세 가지 방식으로 위빙이 가능하다.

- CTW(Compile Time Weaving)
    - Java Compiler를 확장한 형태의 **AJC(AspectJ Compiler)를 통해 컴파일 과정에서 바이트 코드를 조작해 어드바이스 코드를 직접 삽입**
    - 컴파일 과정에서 코드를 조작하는 플러그인과 충돌 가능성이 매우 높음(거의 같이 사용 불가)
- PCW(Post-Complie Weaving = **BW, Binary Weaving**)
    - 컴파일된 바이너리 코드에 기존 클래스 파일과 JAR 파일 위빙(주로 외부 라이브러리 위빙 시 사용)
    - CTW같은 방식으로 처리
- LTW(Load Time Weaving)
    - 클래스로더가 JVM에 클래스를 로딩할 때 바이트 코드를 조작해 어드바이스 코드 삽입
    - 컴파일 시간은 CTW보다 상대적으로 빠르지만, 오브젝트가 메모리에 올라가는 과정에서 위빙이 일어나기 때문에 런타임 시 시간은 CTW보다 상대적으로 느리다.
        - Application Context에 객체가 로드될 때 객체 핸들링이 발생하므로 퍼포머스가 저하됨

## Spring AOP vs AspectJ

![](https://velog.velcdn.com/images/minide/post/04cfeffb-8586-4976-8311-5835a726335f/image.png)

## References

[[10분 테코톡] 콩하나의 스프링 AOP](https://www.youtube.com/watch?v=7BNS6wtcbY8&pp=ygUKc3ByaW5nIGFvcA%3D%3D)

[Proxying Mechanisms](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)

[자바 AOP의 모든 것(Spring AOP & AspectJ)](https://jiwondev.tistory.com/152#head2)

[Spring AOP Weaving, Proxy](https://velog.io/@dnjwm8612/AOP-Weaving-Proxy#spring-aop%EC%99%80-aspectj)

[[Spring] AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)의 이해 - (1/3)](https://mangkyu.tistory.com/161)
