# Reflection

## 1. Reflection

리플렉션(Reflection)은 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 필드, 생성자에 접근할 수 있도록 해주는 자바 API다.

리플렉션은 실행 시간에 동적으로 특정 클래스의 정보를 추출할 수 있다.

리플렉션을 사용하면 접근 제어자와 무관하게 **런타임에 클래스의 메소드를 호출**할 수 있다.

`java.lang.reflect` 패키지 아래에서 리플렉션에 필요한 클래스들을 제공한다.

### 1.1. Reflection을 통해 가져올 수 있는 정보

1. Class
2. Constructor
3. Method
4. Field
5. Enum
6. Annotation
7. Parent Class & Interface
8. etc

### 1.2. Reflection 기능

1. Class 가져오기
    1. `{클래스 타입}.class`

        ```java
        Class<?> clazz = Person.class;
        ```

    2. `{인스턴스}.getClass()`

        ```java
        Person 어정윤 = new Person("어정윤");
        Class<?> clazz = 어정윤.getClass();
        ```

    3. `Class.forName(”{전체 도메인 네임}”)`

        ```java
        Class<?> clazz = Class.forName("org.jeongyuneo.Person");
        ```

2. Class의 메소드 사용
    - `getMethods()`: 상위 클래스와 상위 인터페이스에서 **상속한 메소드를 포함**해 접근 제한자가 `public`인 메소드들을 모두 가져옴
    - `getDeclaredMethods()`: **접근 제한자 상관없이** 상속한 메소드를 제외하고 직접 클래스에서 선언한 메소드들을 모두 가져옴
    - `setAccessible(true)`: 접근 제한자가 `public`이 아닐 경우 접근을 허용해줌

      접근 제한자가 `public`이 아닌 필드나 메소드 등에 접근했을 때 아래와 같은 예외가 발생한다.

      ![](https://velog.velcdn.com/images/minide/post/57e465ae-2f16-4320-9b63-8f19b5f9e70d/image.png)

      따라서, `setAccessible()` 메소드를 이용해 접근 제한자가 `public`이 아닌 경우에도 접근할 수 있음


### 1.3. Reflection 사용

- 프레임워크나 라이브러리 (ex. JPA, Jackson, Mockito, JUnit 등)
    - 리플렉션을 사용하는 프레임워크나 라이브러리에서 객체에 **기본생성자**가 필요한 이유
        - 생성자의 종류가 많은 경우 어떤 생성자를 사용해야 하는지 프레임워크나 라이브러리가 선택하기 어려움
        - 생성자에 로직이 있는 경우 원하는 값을 바로 넣어줄 수 없음
        - 파라미터들의 타입이 같은 경우 필드와 이름이 다르면 값을 알맞게 넣어주기 어려움

      ⇒ 기본 생성자로 객체를 생성하고 필드를 통해 값을 넣어주는 것이 **가장 간단한 방법**이기 때문

- 어노테이션
    - 동작 원리
        1. 리플렉션을 통해 클래스나 메소드, 파라미터 정보를 가져옴
        2. 리플렉션의 `getAnnotations(s)`, `getDeclaredAnnotation(s)` 등의 메소드를 통해 원하는 어노테이션이 붙어 있는지 확인
        3. 어노테이션이 붙어 있으면 원하는 로직 수행

### 1.4. Reflection 단점

- 속도가 느리다.

  일반적으로 메소드 호출 시 컴파일 시점에 분석된 클래스를 사용하지만, 리플렉션은 **런타임 시점**에 클래스를 분석하므로 JVM을 최적화할 수 없어 성능저하가 발생한다.

- 컴파일 시점에 타입 체크가 불가능하다.

  리플렉션을 런타임 시점에 클래스 정보를 알게 되므로 컴파일 시점에 타입 체크가 불가능하다.

- 객체의 추상화가 파괴된다.
- 코드가 지저분하고 장황해진다.

참고: 이펙티브 자바 아이템 65

## References

[[10분 테코톡] 파랑, 아키의 리플렉션](https://www.youtube.com/watch?v=67YdHbPZJn4)

[자바 리플렉션 (Reflection) 기초](https://hudi.blog/java-reflection/)
