# Generics

## 1. Generics

자바에서 제네릭(Generics)은 **클래스 내부에서 사용할 데이터 타입을 외부에서 지정**하는 기법이다. 컴파일 타임에 타입을 체크함으로써 코드의 안정성을 높여준다.

## 2. Generics 사용 이유

1. **컴파일 타임**에 강력한 타입 검사 → 예외 방지

    ```java
    @Test
    void 제네릭_미사용() {
        List stringList = new ArrayList<>();
        stringList.add("jeongyuneo");
        stringList.add(1);
        String result = (String) stringList.get(0) + (String) stringList.get(1);    // Runtime Error
    }
    
    @Test
    void 제네릭_사용() {
        List<String> stringList = new ArrayList<>();
        stringList.add("jeongyuneo");
        stringList.add(1);  // Compile Error
    }
    ```

2. **타입 변환(캐스팅) 제거** → 성능 향상

    ```java
    @Test
    void 제네릭_미사용() {
        List stringList = new ArrayList<>();
        stringList.add("jeongyuneo");
        String result = (String) stringList.get(0);
    }
    
    @Test
    void 제네릭_사용() {
        List<String> stringList = new ArrayList<>();
        stringList.add("jeongyuneo");
        String result = stringList.get(0);
    }
    ```


## 3. Generics Type Parameter

제네릭은 꺽쇠 기호(`<>`)를 사용하는데 이를 다이아몬드 연산자라고 한다.

다이아몬드 연산자 안에 식별자 기호를 지정함으로써 파라미터화 할 수 있는데, 이를 제네릭의 **타입 매개변수 혹은 타입 변수**라고 한다.

![](https://velog.velcdn.com/images/minide/post/4122f11e-9c9a-4ae9-b2bb-8b2296dd56c2/image.png)

타입 매개변수는 **제네릭을 이용한 클래스나 메소드를 설계**할 때 사용된다.

```java
// 제네릭 클래스 정의
class Bookshelf<T> {

    private List<T> books = new ArrayList<>();

    public void add(T book) {
        books.add(book);
    }
}

// 제네릭 클래스 사용
Bookshelf<Integer> intBookshelf = new Bookshelf<>();  // 제네릭 타입 매개변수에 정수 타입 할당
Bookshelf<String> stringBookshelf = new Bookshelf<>();  // 제네릭 타입 매개변수에 문자열 타입 할당
Bookshelf<Novel> novelBookshelf = new Bookshelf<>();  // 제네릭 타입 매개변수에 클래스 타입 할당
```

제네릭 클래스의 `<T>` 부분은 실행부에서 타입을 받아와 내부에서 `T` 타입으로 지정한 멤버들에게 전파하여 타입이 구체적으로 설정된다. 이를 **구체화(Specialization)**라고 한다.

```java
// 제네릭 클래스 선언
Bookshelf<Novel> novelBookshelf = new Bookshelf<>();

// 타입 매개변수 타입 전파
class Bookshelf<Novel> {

    private List<Novel> books = new ArrayList<>();

    public void add(Novel book) {
        books.add(book);
    }
}
```

JDK 1.7 이후부터 **생성자 부분의 제네릭 타입을 생략할 수 있다.** 제네릭에서 타입 추론을 통해 생략된 곳에 타입을 넣어주기 때문에 생략해도 문제되지 않는다.

제네릭에 **할당 가능한 타입은 Reference 타입 뿐**이다. 즉, int형이나 double형과 같은 Primitive 타입은 제네릭 타입 파라미터로 넘길 수 없다. Wrapper 클래스를 사용함으로써 **다형성**의 원리가 적용되어 업캐스팅을 통해 타입 파라미터로 할당된 객체의 자식 객체도 할당이 가능하다.

```java
class Book { }
class Novel extends Book { }
class Essay extends Book { }

class Bookshelf<T> {

    private List<T> books = new ArrayList<>();

    public void add(T book) {
        books.add(book);
    }
}

public class Main {

    public static void main(String[] args) {
        Bookshelf<Book> bookshelf = new Bookshelf<>();
        bookshelf.add(new Book());
        bookshelf.add(new Novel());
        bookshelf.add(new Essay());
}
```

제네릭은 **복수 타입 파라미터를 허용**한다. 제네릭 타입의 구분은 쉼표(,)로 하며 `<T, U>`와 같은 형식으로 복수 타입 파라미터를 지정할 수 있다.

```java
class Bookshelf<T, U> {

    private List<T> novels = new ArrayList<>();
		private List<U> essays = new ArrayList<>();

    public void add(T novel, U essay) {
        novels.add(novel);
        essays.add(essay);
    }
}

public class Main {

    public static void main(String[] args) {
        Bookshelf<Novel, Essay> bookshelf = new Bookshelf<>();
        bookshelf.add(new Novel(), new Essay());
    }
}
```

또한 제네릭 객체를 제네릭 타입 파라미터로 받을 수 있다.

```java
List<List<String>> list = new ArrayList<List<String>>();
```

제네릭 타입 파라미터의 네이밍은 문법적으로 정해져있지 않아 자유롭게 네이밍이 가능하다. 다만, 개발의 용이함을 위해 통상적으로 사용하는 아래와 같은 컨벤션이 존재한다.

![](https://velog.velcdn.com/images/minide/post/f74ab0fe-d4f3-4f12-9e4a-1784f0ba6084/image.png)

## 4. Generics Wildcards

제네릭 **서브 타입 간에는 형변환이 불가능**하다. 즉, 전달받은 타입으로만 캐스팅이 가능하다. 위에서 말했던 포함 원소로서의 다형성과 헷갈리면 안된다.

```java
List<Object> list = new ArrayList<Integer>();  // 불가능
```

따라서 제네릭 간의 형변환을 하기 위해서는 제네릭에서 제공하는 와일드 카드(`?`)를 사용해야 한다.

- Unbounded Wildcards `<?>`
    - 제한 없음
    - 타입 파라미터를 대치하는 구체적인 타입으로 모든 클래스나 인터페이스 타입이 올 수 있음
- Upper Bounded Wildcards `<? extends 상위타입>`
    - 상위 클래스 제한
    - 타입 파라미터를 대치하는 구체적인 타입으로 상위 타입이나 상위 타입의 하위 타입만 올 수 있음
- Lower Bounded Wildcards `<? super 하위타입>`
    - 하위 클래스 제한
    - 타입 파라미터를 대치하는 구체적으로 타입으로 하위 타입이나 하위 타입의 상위 타입만 올 수 있음

### 4.1. Variance

변성(Variance)은 타입 계층 관계에서 서로 다른 타입 간에 어떤 관계가 있는지 나타내는 개념이다.

- 무공변(Invariance)
    - 타입 B가 타입 A의 하위 타입일 때, Category<B>가 Category<A>의 하위 타입이 아닌 경우(아무 관계 없음)
    - ex. 제네릭 `<T>`
- 공변(Covariance)
    - 타입 B가 타입 A의 하위 타입일 때, Category<B>가 Category<A>의 하위 타입인 경우
    - ex. 배열, 제네릭 `<? extends T>`
- 반공변(Contravariance)
    - 타입 B가 타입 A의 하위 타입일 때, Category<B>가 Category<A>의 상위 타입인 경우
    - ex. 제네릭 `<? super T>`

```java
// 공변
Object[] array = new Integer[1];

// 무공변
List<Object> list = new ArrayList<Integer<>();  // Compile Error
```

### 4.2. PECS

**producer-extends / consumer-super**

이펙티브 자바에서 소개된 개념으로, 와일드 카드 상한/하한 제한 시 생성하는 곳에서는 extends를, 소비를 하는 곳에서는 super를 사용하라는 공식

## 5. Generics Type Erasure

제네릭은 JDK 1.5에서 등장했기 때문에 JDK 1.5 이전 버전에서도 동작할 수 있도록 하기 위해 컴파일 타임에 타입 검사 후 런타임에는 타입을 소거하는 전략을 선택했다.

1. 타입 매개변수의 경계가 없는 경우, **Object**로 타입 파라미터 변경
2. 타입 매개변수의 경계가 있는 경우, **경계 타입**으로 타입 파라미터 변경
3. 타입 안정성을 유지하기 위해 필요한 경우 **타입 변환 추가**
4. 제네릭 타입을 상속받은 클래스의 다형성을 유지하기 위해 **Bridge Method** 생성

## 6. Generics 사용 시 주의사항

1. 제네릭 타입의 객체 생성 불가

    ```java
    class Sample<T> {
    
        public void generateGenericInstance() {
            T t = new T();  // 불가능
        }
    }
    ```

2. `static` 멤버에 제네릭 타입 올 수 없음

   `static` 멤버는 제네릭 객체가 생성되기 전에 자료 타입이 정해져야 하기 때문에 논리적 오류가 발생한다.

3. 제네릭으로 배열 선언 주의

   제네릭 클래스의 자체를 배열로 만들 수는 없지만, 제네릭 타입의 배열 선언은 가능하다.

    ```java
    class Sample<T> { }
    
    Sample<Integer>[] arr1 = new Sample<>[10];  // 불가능
    Sample<Integer>[] arr1 = new Sample[10];  // 가능
    ```


## References

[자바 제네릭(Generics) 개념 & 문법 정복하기 - Inpa Dev](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%ADGenerics-%EA%B0%9C%EB%85%90-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)
